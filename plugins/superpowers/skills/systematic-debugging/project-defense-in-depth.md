# 纵深防御验证（servergo 项目版）

## 概述

在 servergo 这类多服务、Actor 模型、涉及真实资产的游戏服务器中，修复一个 bug 时只在一个地方添加验证往往不够。不同代码路径、重构、mock 测试或异步任务都可能绕过单一检查。

**核心原则：** 在数据经过的每一层都进行验证。让该 bug 在结构上不可能发生。

## 为什么需要多层验证

单一验证：“我们修好了这个 bug”  
多层验证：“我们让该 bug 不可能发生”

不同层捕获不同情况：
- **入口验证** 捕获非法路由、错误协议、越界参数
- **业务逻辑验证** 捕获状态机违规、资产不足、分数异常
- **环境防护** 阻止测试/生产中的危险操作（如误删数据、在源码目录执行命令）
- **调试探针** 在其他层失效时提供取证线索

## 四层验证

### 第 1 层：入口点验证

**目的：** 在 API / 协议边界拒绝明显无效的输入。

```go
// Gate / 子游戏路由入口示例
func (c *Core) onSomeRequest(player *game.Player, req *pb.SomeReq) error {
    if player == nil || player.IsOffline() {
        return code.PlayerNotFound.Err()
    }
    if req.GetRoomId() == "" {
        return code.InvalidParameter.Err()
    }
    if req.GetAmount() <= 0 {
        return code.InvalidParameter.Err()
    }
    // ... 继续执行
}
```

```go
// gRPC 服务入口示例
func (s *Server) ChangeGold(ctx context.Context, args *assetspb.ChangeGoldArgs, reply *assetspb.ChangeGoldReply) error {
    if args.GetUid() <= 0 {
        reply.Code = int32(code.InvalidParameter.Code())
        return nil
    }
    if args.GetDelta() == 0 {
        reply.Code = 0
        return nil
    }
    // ... 继续执行
}
```

### 第 2 层：业务逻辑验证

**目的：** 确保数据对该操作有意义，状态机允许该操作。

```go
// 牌桌业务逻辑示例
func (tl *TableLogic) onBet(player *game.Player, req *pb.BetReq) error {
    table := player.Table()
    if table == nil {
        return code.NotInTable.Err()
    }
    if table.Status() != game.Playing {
        return code.TableStatusNotAllowed.Err()
    }
    if !table.IsCurrentActor(player.UID()) {
        return code.NotYourTurn.Err()
    }
    if req.GetScore() > player.Score() {
        return code.ScoreNotEnough.Err()
    }
    // ... 继续执行
}
```

```go
// 结算前业务校验示例
func (tl *TableLogic) settle() error {
    for _, r := range results {
        if r.GetScore() == 0 {
            continue
        }
        if r.GetUid() <= 0 {
            return code.InvalidPlayer.Err()
        }
    }
    // 赢分总和应等于输分总和（零和校验）
    var total int64
    for _, r := range results {
        total += r.GetScore()
    }
    if total != 0 {
        return code.SettlementNotZeroSum.Err()
    }
    // ... 继续执行
}
```

### 第 3 层：环境防护

**目的：** 在特定上下文中阻止危险操作。

```go
// 测试环境防止误操作生产数据库/Redis
func flushTestData() error {
    if os.Getenv("DUE_ETC") == "" {
        return errors.New("DUE_ETC not set, refusing to flush data")
    }
    if !strings.Contains(os.Getenv("DUE_ETC"), "test") {
        return errors.New("DUE_ETC does not point to test config, refusing to flush")
    }
    // ... 执行清理
}
```

```go
// 防止在源码目录执行危险命令
func runGitInit(dir string) error {
    cwd, _ := os.Getwd()
    absDir, _ := filepath.Abs(dir)
    absCwd, _ := filepath.Abs(cwd)

    if strings.HasPrefix(absDir, absCwd) && !strings.Contains(absDir, "tmp") {
        return fmt.Errorf("refusing git init inside source tree: %s", absDir)
    }
    // ... 继续执行
}
```

```go
// 生产环境禁止某些调试接口
func (w *Web) onDebugReset(ctx *fiber.Ctx) error {
    if os.Getenv("APP_ENV") == "production" {
        return fiber.NewError(fiber.StatusForbidden, "debug reset disabled in production")
    }
    // ... 继续执行
}
```

### 第 4 层：调试探针

**目的：** 为取证捕获上下文，尤其是在前面所有层都失效时。

```go
package main

import (
    "context"
    "runtime/debug"

    "github.com/dobyte/due/v2/log"
)

// 在资产变更前记录
func changeGold(ctx context.Context, uid int64, delta int64, reason string) error {
    log.Errorf("DEBUG changeGold: uid=%d delta=%d reason=%s stack=%s",
        uid, delta, reason, debug.Stack())

    // ... 实际业务调用
}
```

```go
// 在 table.Finish 前记录
func (tl *TableLogic) onGameOver() {
    log.Errorf("DEBUG game over: tableID=%s round=%d results=%+v stack=%s",
        tl.table.ID(), tl.table.Round(), tl.results, debug.Stack())

    tl.table.Finish(tl.doSettle)
}
```

## 应用模式

发现 bug 时：

1. **追踪数据流** - 错误值起源于哪里？在哪里被使用？（参见 `root-cause-tracing-go.md`）
2. **列出所有检查点** - 数据经过的每一个点：Gate → Game Actor → Mesh → Redis → Mongo
3. **在每一层添加验证** - 入口、业务、环境、调试
4. **测试每一层** - 尝试绕过第 1 层，验证第 2 层能否捕获；mock 绕过第 2 层时，第 3 层应兜底

## 来自 servergo 的示例

### Bug：空的 `tableID` 导致 tabledesk 异常

**数据流：**
1. 客户端发送协议时 `tableID` 为空
2. Gate 转发到 Game node
3. Game Actor 处理消息
4. 调用 `syncPlayersToDeskAsync` 传入空 `tableID`
5. tabledesk 服务出现空 key

**添加的四层验证：**
- 第 1 层：Gate/Game 路由入口校验 `tableID` 非空
- 第 2 层：`syncPlayersToDeskAsync` 调用前校验 `tableID`
- 第 3 层：tabledesk 服务对空 `tableID` 直接返回 `InvalidParameter`
- 第 4 层：在同步前记录 `tableID`、玩家列表与堆栈

### Bug：结算分数符号错误导致玩家资产扣反

**数据流：**
1. 子游戏构造 `PlayerResult.Score`
2. `table.Finish` 调用资产处理器
3. `goldAssetHandler.Settle` 调用 `ChangeGold`
4. Redis/Mongo 资产被错误扣减

**添加的四层验证：**
- 第 1 层：子游戏路由入口校验分数范围
- 第 2 层：`table.Finish` 前进行零和校验（所有 `Score` 之和为 0）
- 第 3 层：资产服务在测试环境拒绝单笔超过阈值的变更
- 第 4 层：在 `ChangeGold` 前记录 `uid/delta/reason/stack`

### Bug：Actor 消息处理阻塞导致牌桌卡死

**数据流：**
1. `OnLeaveTable` 在 Actor goroutine 中执行
2. 内部调用同步 RPC（如 club 服务）
3. RPC 超时阻塞 Actor 消息队列
4. 后续所有消息排队，牌桌无法推进

**添加的四层验证：**
- 第 1 层：代码审查禁止在 `TableLifecycle` / `SeatLogic` 回调中直接 RPC
- 第 2 层：在 Actor 消息分发处检测 handler 执行耗时，超过阈值告警
- 第 3 层：RPC 客户端统一设置较短超时，避免无限阻塞
- 第 4 层：在 Actor 消息入口/出口记录消息类型与耗时

## 关键洞察

四层验证都是必要的。在 servergo 的实践中，每一层都捕获过其他层遗漏的 bug：
- 不同客户端协议绕过了入口校验
- 测试 mock 绕过了业务逻辑检查
- 多平台路径差异需要环境防护
- 调试日志识别出异步任务中的结构性误用

**不要只停留在一个验证点。** 在 Gate、Game Actor、Mesh 服务、Redis/Mongo 访问的每一层都添加检查。

## 项目实践检查清单

- [ ] 所有 gRPC / RPCX 服务入口校验 `uid > 0`、`tableID != ""` 等基础参数
- [ ] 所有子游戏协议 handler 校验玩家在线、在桌、状态合法
- [ ] `table.Finish` 前校验分数零和、玩家存在、资产类型合法
- [ ] 资产变更前校验余额/积分充足（`CheckBalance`）
- [ ] Redis 锁 key 包含正确维度（uid / assetType），避免作用域过大
- [ ] 测试/生产环境敏感操作有明确的环境检查（`DUE_ETC`、`APP_ENV`）
- [ ] 危险操作前使用 `log.Errorf` + `debug.Stack()` 记录上下文
- [ ] 新增校验后补充测试用例，验证该层能否独立捕获异常
