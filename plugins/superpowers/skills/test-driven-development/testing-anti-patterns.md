# 测试反模式

**在以下情况加载本参考：** 编写或修改测试、添加 mock，或忍不住想要向生产代码添加仅测试用的方法时。

## 概述

测试必须验证真实行为，而不是 mock 行为。Mock 是用于隔离的手段，不是被测试的对象。

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
```typescript
// ❌ BAD: 测试 mock 是否存在
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么错误：**
- 你在验证 mock 是否存在，而不是组件是否工作
- mock 存在时测试通过，不存在时失败
- 对真实行为一无所知

**你的人类伙伴会纠正：** “我们是在测试 mock 的行为吗？”

**修复：**
```typescript
// ✅ GOOD: 测试真实组件，或不使用 mock
test('renders sidebar', () => {
  render(<Page />);  // 不要 mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// 或者如果为了隔离必须 mock sidebar：
// 不要断言 mock - 测试 Page 在 sidebar 存在时的行为
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
```typescript
// ❌ BAD: 仅用于测试的 destroy()
class Session {
  async destroy() {  // 看起来像是生产 API！
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// 在测试中
afterEach(() => session.destroy());
```

**为什么错误：**
- 生产类被仅测试用的代码污染
- 如果在生产环境中误调用会有危险
- 违反 YAGNI 和单一职责原则
- 混淆对象生命周期与实体生命周期

**修复：**
```typescript
// ✅ GOOD: 测试工具处理测试清理
// Session 没有 destroy() - 它在生产中是无状态的

// 在 test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// 在测试中
afterEach(() => cleanupSession(session));
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
```typescript
// ❌ BAD: Mock 破坏了测试逻辑
test('detects duplicate server', () => {
  // Mock 阻止了测试依赖的 config 写入！
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // 应该抛出 - 但不会！
});
```

**为什么错误：**
- 被 mock 的方法具有测试依赖的副作用（写入配置）
- 为了“保险”而过度 mock 会破坏实际行为
- 测试因错误原因通过或神秘地失败

**修复：**
```typescript
// ✅ GOOD: 在正确的层级 mock
test('detects duplicate server', () => {
  // 只 mock 慢速的服务器启动
  vi.mock('MCPServerManager'); 

  await addServer(config);  // 配置被写入
  await addServer(config);  // 检测到重复 ✓
});
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
```typescript
// ❌ BAD: 部分 mock - 只有你以为需要的字段
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // 缺少：下游代码使用的 metadata
};

// 之后：当代码访问 response.metadata.requestId 时会崩溃
```

**为什么错误：**
- **部分 mock 隐藏了结构假设** - 你只 mock 了已知的字段
- **下游代码可能依赖你未包含的字段** - 静默失败
- **测试通过但集成失败** - mock 不完整，真实 API 完整
- **虚假信心** - 测试无法证明真实行为

**铁律：** Mock 完整的数据结构，就像它在现实中存在的那样，而不是只 mock 当前测试使用的字段。

**修复：**
```typescript
// ✅ GOOD: 镜像真实 API 的完整性
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // 真实 API 返回的所有字段
};
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

## 当 Mock 变得过于复杂

**警告信号：**
- Mock 设置比测试逻辑还长
- Mock 所有东西来让测试通过
- Mock 缺少真实组件的方法
- Mock 变更时测试崩溃

**你的人类伙伴会问：** “我们这里需要使用 mock 吗？”

**考虑：** 使用真实组件的集成测试通常比复杂 mock 更简单。

## TDD 预防这些反模式

**为什么 TDD 有帮助：**
1. **先写测试** → 迫使你思考到底在测试什么
2. **观察它失败** → 确认测试测试的是真实行为，而不是 mock
3. **最小化实现** → 不会有仅测试用的方法混入
4. **真实依赖** → 在 mock 之前你能看到测试真正需要什么

**如果你在测试 mock 行为，你就违反了 TDD** - 你在没有先让测试针对真实代码失败的情况下添加了 mock。

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

- 断言检查 `*-mock` 测试 ID
- 只在测试文件中调用的方法
- Mock 设置占测试的 50% 以上
- 移除 mock 后测试失败
- 无法解释为什么需要 mock
- “为了保险而 mock”

## 底线

**Mock 是隔离工具，不是测试对象。**

如果 TDD 揭示你在测试 mock 行为，那你就走错了。

修复：测试真实行为，或质疑你为什么要 mock。
