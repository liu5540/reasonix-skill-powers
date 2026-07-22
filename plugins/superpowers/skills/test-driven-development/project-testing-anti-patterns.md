# 测试反模式（servergo 项目版）

**在以下情况加载本参考：** 编写或修改测试、添加 mock，或忍不住想要向生产代码添加仅测试用的方法时。

## 概述

测试必须验证真实行为，而不是 mock 行为。Mock 是用于隔离的手段，不是被测试的对象。

servergo 项目明确约定：**测试使用真实代码（仅不可避免时用 mock）**。这与本技能完全一致。

**核心原则：** 测试代码做什么，而不是 mock 做什么。

**严格遵循 TDD 可以预防这些反模式。**

## 铁律

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## 反模式 1：测试 Mock 行为

**违规：**
```go
// ❌ BAD: 测试 mock 是否被调用
func TestTableFinishCallsSettle(t *testing.T) {
    mockSettle := &mockSettleFunc{}
    table := NewTable(/* ... */)
    table.Finish(mockSettle.fn)

    // 你在验证 mock 被调用了多少次，而不是结算是否正确
    require.True(t, mockSettle.called)
}
```

**为什么错误：**
- 你在验证 mock 是否被调用，而不是真实结算逻辑
- mock 被调用时测试通过，但生产代码可能传错参数
- 对真实行为一无所知

**你的人类伙伴会纠正：** “我们是在测试 mock 的行为吗？”

**修复：**
```go
// ✅ GOOD: 测试真实结算结果
func TestTableFinishUpdatesPlayerAsset(t *testing.T) {
    // 使用真实玩家、真实资产处理器（或内存版测试 double）
    table := createTestTableWithPlayers(t, []int64{10086, 10087})
    before := table.GetPlayer(10086).Score()

    table.Finish(func() {
        // 真实结算逻辑：玩家 10086 赢 100 分
        table.GetPlayer(10086).AddScore(100)
    })

    after := table.GetPlayer(10086).Score()
    require.Equal(t, before+100, after)
}
```

### Gate Function

```
BEFORE asserting on any mock element:
  Ask: "Am I testing real component behavior or just mock existence?"

  IF testing mock existence:
    STOP - Delete the assertion or unmock the component

  Test real behavior instead
```

## 反模式 2：生产代码中的仅测试方法

**违规：**
```go
// ❌ BAD: 仅用于测试的 ResetState()
type Table struct {
    state game.TableStatus
    // ...
}

// ResetState 看起来像是生产 API！
func (t *Table) ResetState() {
    t.state = game.Idle
    t.players = nil
}

// 在测试中
func TestTableLifecycle(t *testing.T) {
    table := NewTable(...)
    t.Cleanup(func() { table.ResetState() })
}
```

**为什么错误：**
- 生产类被仅测试用的代码污染
- 如果在生产环境中误调用会有危险（如把正在游戏的桌子重置）
- 违反 YAGNI 和单一职责原则
- 混淆对象生命周期与实体生命周期

**修复：**
```go
// ✅ GOOD: 生产代码没有 ResetState，测试工具处理清理

// internal/game/table_test.go 或 testutil 包
func cleanupTable(t *testing.T, table *game.Table) {
    t.Helper()
    // 通过生产暴露的合法生命周期接口清理
    if table.Status() != game.Over {
        table.Finish(nil)
    }
    table.Destroy()
}

// 在测试中
func TestTableLifecycle(t *testing.T) {
    table := NewTable(...)
    t.Cleanup(func() { cleanupTable(t, table) })
}
```

### Gate Function

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?"

  IF yes:
    STOP - Don't add it
    Put it in test utilities instead

  Ask: "Does this class own this resource's lifecycle?"

  IF no:
    STOP - Wrong class for this method
```

## 反模式 3：在不理解依赖的情况下 Mock

**违规：**
```go
// ❌ BAD: Mock 破坏了测试逻辑
func TestCreateRoomDetectsDuplicate(t *testing.T) {
    // Mock 阻止了测试依赖的 Redis 写入！
    mockRedis := &mockRedis{setFunc: func(key string, val any) error { return nil }}
    mgr := NewManager(WithRedis(mockRedis))

    err1 := mgr.CreateRoom(ctx, roomConfig)
    err2 := mgr.CreateRoom(ctx, roomConfig) // 应该返回重复错误，但不会！

    require.NoError(t, err1)
    require.NoError(t, err2) // ❌ 错误地通过了
}
```

**为什么错误：**
- 被 mock 的方法具有测试依赖的副作用（写入 Redis）
- 为了“保险”而过度 mock 会破坏实际行为
- 测试因错误原因通过或神秘地失败

**修复：**
```go
// ✅ GOOD: 在正确的层级隔离
func TestCreateRoomDetectsDuplicate(t *testing.T) {
    // 使用真实 Redis（测试专用 DB），或只 mock 真正慢的外部 HTTP 调用
    rdb := testutil.NewTestRedis(t)
    mgr := NewManager(WithRedis(rdb))

    err1 := mgr.CreateRoom(ctx, roomConfig)
    err2 := mgr.CreateRoom(ctx, roomConfig)

    require.NoError(t, err1)
    require.ErrorIs(t, err2, code.RoomAlreadyExists) // ✅ 正确检测到重复
}
```

### Gate Function

```
BEFORE mocking any method:
  STOP - Don't mock yet

  1. Ask: "What side effects does the real method have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (the actual slow/external operation)
    OR use test doubles that preserve necessary behavior
    NOT the high-level method the test depends on

  IF unsure what test depends on:
    Run test with real implementation FIRST
    Observe what actually needs to happen
    THEN add minimal mocking at the right level

  Red flags:
    - "I'll mock this to be safe"
    - "This might be slow, better mock it"
    - Mocking without understanding the dependency chain
```

## 反模式 4：不完整的 Mock

**违规：**
```go
// ❌ BAD: 部分 mock - 只有你以为需要的字段
mockResponse := &assetspb.FetchAssetsReply{
    Gold: 1000,
    // 缺少：下游代码使用的 SafeBoxGold
}

mockAssetsClient := &mockAssetsClient{}
mockAssetsClient.On("FetchAssets", mock.Anything, mock.Anything).Return(mockResponse, nil)

// 之后：当代码访问 reply.SafeBoxGold 时会使用零值，导致错误逻辑
```

**为什么错误：**
- **部分 mock 隐藏了结构假设** - 你只 mock 了已知的字段
- **下游代码可能依赖你未包含的字段** - 静默失败
- **测试通过但集成失败** - mock 不完整，真实 API 完整
- **虚假信心** - 测试无法证明真实行为

**铁律：** Mock 完整的数据结构，就像它在现实中存在的那样，而不是只 mock 当前测试使用的字段。

**修复：**
```go
// ✅ GOOD: 镜像真实 API 的完整性
mockResponse := &assetspb.FetchAssetsReply{
    Gold:        1000,
    SafeBoxGold: 500,
    Diamond:     0,
    Score:       0,
    // 真实 API 返回的所有字段
}

// 更好的做法：使用真实 gRPC 服务或生成器
reply := testutil.NewFetchAssetsReply(t, testutil.WithGold(1000))
```

### Gate Function

```
BEFORE creating mock responses:
  Check: "What fields does the real API response contain?"

  Actions:
    1. Examine actual API response from docs/examples
    2. Include ALL fields system might consume downstream
    3. Verify mock matches real response schema completely

  Critical:
    If you're creating a mock, you must understand the ENTIRE structure
    Partial mocks fail silently when code depends on omitted fields

  If uncertain: Include all documented fields
```

## 反模式 5：把集成测试当作事后补充

**违规：**
```
✅ Implementation complete
❌ No tests written
"Ready for testing"
```

**为什么错误：**
- 测试是实现的一部分，不是可选的后续步骤
- TDD 本可以抓住这一点
- 没有测试就不能声称完成

**修复：**
```
TDD cycle:
1. Write failing test
2. Implement to pass
3. Refactor
4. THEN claim complete
```

在 servergo 中的体现：
```bash
# ❌ 错误流程
写 game/laoyancai 结算逻辑 → 手动跑 E2E → "完成了"

# ✅ 正确流程
写 game/laoyancai/app/logic/table_logic_test.go 失败测试
→ 实现结算逻辑
→ 跑 go test ./game/laoyancai/...
→ 跑 client/runner.exe --case laoyancai/settlement_flow
→ 重构
→ 标记完成
```

## 当 Mock 变得过于复杂

**警告信号：**
- Mock 设置比测试逻辑还长
- Mock 所有东西来让测试通过
- Mock 缺少真实组件的方法
- Mock 变更时测试崩溃

**你的人类伙伴会问：** “我们这里需要使用 mock 吗？”

**考虑：** 使用真实组件的集成测试通常比复杂 mock 更简单。

### servergo 中的真实替代方案

| 依赖 | 不要这样做 | 考虑这样做 |
|------|-----------|-----------|
| Redis | mock `RedisClient` 所有方法 | 启动测试 Redis 容器或本地实例，使用独立 DB |
| MongoDB | mock DAO | 使用真实 MongoDB 测试库，测试后清理集合 |
| gRPC 服务 | mock 整个 client | 启动真实服务进程或内存服务 |
| Actor/Table | mock Table 所有方法 | 使用真实 Table 并控制生命周期 |
| 时间 | mock `time.Now` | 使用依赖注入的 `clock.Clock` 接口 |

## TDD 预防这些反模式

**为什么 TDD 有帮助：**
1. **先写测试** → 迫使你思考到底在测试什么
2. **观察它失败** → 确认测试测试的是真实行为，而不是 mock
3. **最小化实现** → 不会有仅测试用的方法混入
4. **真实依赖** → 在 mock 之前你能看到测试真正需要什么

**如果你在测试 mock 行为，你就违反了 TDD** - 你在没有先让测试针对真实代码失败的情况下添加了 mock。

## servergo 测试实践

### 项目约定

- 测试使用真实代码，仅不可避免时用 mock
- 单元测试：`go test ./internal/...`（需 `DUE_ETC`）
- E2E 测试：`client/runner.exe --case <game>/<case>`
- 不要在生产代码里加 `// only for test` 的方法
- 不要修改 `*.pb.go` 或 `dao/<name>/internal/` 来迁就测试

### 推荐结构

```go
func TestSomeBehavior(t *testing.T) {
    // Arrange：使用真实依赖或最小化 test double
    ctx := context.Background()
    rdb := testutil.NewTestRedis(t)
    svc := NewService(rdb)

    // Act
    err := svc.DoSomething(ctx, args)

    // Assert：验证真实行为
    require.NoError(t, err)
    got, err := rdb.Get(ctx, "key").Result()
    require.NoError(t, err)
    require.Equal(t, "expected", got)
}
```

## 快速参考

| 反模式 | 修复 |
|--------------|-----|
| 断言 mock 元素 | 测试真实组件或取消 mock |
| 生产代码中的仅测试方法 | 移到测试工具中 |
| 不理解就 mock | 先理解依赖，最小化 mock |
| 不完整 mock | 完全镜像真实 API |
| 测试作为事后补充 | TDD - 测试先行 |
| 过度复杂的 mock | 考虑集成测试 |

## 危险信号

- 断言检查 `mock.*` 调用次数
- 只在测试文件中调用的方法
- Mock 设置占测试的 50% 以上
- 移除 mock 后测试失败
- 无法解释为什么需要 mock
- “为了保险而 mock”
- 测试里出现 `// only for test` 的注释

## 底线

**Mock 是隔离工具，不是测试对象。**

如果 TDD 揭示你在测试 mock 行为，那你就走错了。

修复：测试真实行为，或质疑你为什么要 mock。
