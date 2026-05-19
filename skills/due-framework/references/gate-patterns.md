# due 网关开发模式

本文档介绍 `github.com/dobyte/due/v2` 框架中 Gate（网关）服务的开发模式。

---

## 1. Gate 概述

Gate 服务是客户端与游戏服务器之间的入口，负责：
- 客户端连接管理（TCP / KCP / WebSocket）
- 消息协议编解码
- 会话管理
- 消息路由到 Node

---

## 2. 协议支持

due 支持三种主流协议：

### 2.1 WebSocket 协议（最常用）

适用于 H5 游戏、即时通讯等场景：

```go
package main

import (
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/network/ws/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/gate"
)

func main() {
    container := due.NewContainer()
    server := ws.NewServer(
        ws.WithPort(8800),
        ws.WithMaxConnNum(10000),
    )
    locator := redis.NewLocator()
    registry := etcd.NewRegistry()
    component := gate.NewGate(
        gate.WithServer(server),
        gate.WithLocator(locator),
        gate.WithRegistry(registry),
    )
    container.Add(component)
    container.Serve()
}
```

### 2.2 TCP 协议

适用于对实时性要求高的游戏：

```go
package main

import (
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/network/tcp/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/gate"
)

func main() {
    container := due.NewContainer()
    server := tcp.NewServer(
        tcp.WithPort(9000),
        tcp.WithMaxConnNum(10000),
    )
    locator := redis.NewLocator()
    registry := etcd.NewRegistry()
    component := gate.NewGate(
        gate.WithServer(server),
        gate.WithLocator(locator),
        gate.WithRegistry(registry),
    )
    container.Add(component)
    container.Serve()
}
```

#### TCP 配置选项

```go
server := tcp.NewServer(
    tcp.WithPort(9000),             // 监听端口
    tcp.WithMaxConnNum(10000),      // 最大连接数
    tcp.WithMsgSize(4096),          // 最大消息大小（字节）
    tcp.WithSendChanSize(1024),     // 发送缓冲区大小
    tcp.WithHeartbeatInterval(30),  // 心跳间隔（秒）
    tcp.WithHeartbeatHandler(heartbeatHandler), // 心跳处理器
)
```

### 2.3 KCP 协议

适用于弱网络环境下的实时游戏（MOBA、吃鸡等）：

```go
package main

import (
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/network/kcp/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/gate"
)

func main() {
    container := due.NewContainer()
    server := kcp.NewServer(
        kcp.WithPort(10000),
        kcp.WithMode(0),  // 0: 快速，1: 正常，2: 流畅
    )
    locator := redis.NewLocator()
    registry := etcd.NewRegistry()
    component := gate.NewGate(
        gate.WithServer(server),
        gate.WithLocator(locator),
        gate.WithRegistry(registry),
    )
    container.Add(component)
    container.Serve()
}
```

#### KCP 模式说明

| 模式 | 值 | RTT 目标 | 适用场景 |
|------|---|---------|---------|
| 快速 | 0 | 20ms | 竞技游戏、音游 |
| 正常 | 1 | 40ms | MOBA、FPS |
| 流畅 | 2 | 60ms | MMO、休闲游戏 |

#### KCP 选择建议

- **MOBA/FPS**：推荐模式 1（正常），平衡延迟和带宽
- **音游/格斗**：推荐模式 0（快速），最低延迟
- **MMORPG**：推荐模式 2（流畅），节省带宽
- **卡牌/回合制**：建议使用 WebSocket，KCP 优势不明显

---

## 3. 会话管理

### 3.1 基础 Session 操作

Session 由框架自动管理，通过 `ctx.Session()` 获取：

```go
func handler(ctx gate.Context) {
    session := ctx.Session()
    cid := session.Cid()      // 连接 ID
    uid := session.Uid()      // 绑定的 UID
    session.Set("key", "value")  // 设置 Session 数据
    value := session.Get("key")  // 获取 Session 数据
    session.Del("key")        // 删除 Session 数据
}
```

### 3.2 玩家绑定（UID Binding）

将玩家 UID 绑定到 Session 是核心操作：

```go
package handler

import (
    "github.com/dobyte/due/v2/cluster/gate"
)

type loginReq struct {
    UID   int64  `json:"uid"`
    Token string `json:"token"`
}

type loginRes struct {
    Code int    `json:"code"`
    Msg  string `json:"msg"`
}

func loginHandler(ctx gate.Context) {
    req := &loginReq{}
    res := &loginRes{}
    defer func() { ctx.Response(res) }()

    if err := ctx.Parse(req); err != nil {
        res.Code = code.IllegalRequest.Code()
        return
    }

    // 验证 token 有效性（业务逻辑）

    // 将 UID 绑定到 Session
    if err := ctx.Session().Bind(req.UID); err != nil {
        res.Code = code.InternalError.Code()
        return
    }

    res.Code = code.OK.Code()
    res.Msg = "登录成功"
}
```

### 3.3 玩家推送消息（Push by UID）

绑定 UID 后，可以通过 UID 向特定玩家推送消息：

```go
import (
    "context"
    "github.com/dobyte/due/v2/cluster/gate"
)

// SendToUser 向指定玩家推送消息
func SendToUser(ctx context.Context, proxy *gate.Proxy, uid int64, route int32, data interface{}) error {
    return proxy.Push(ctx, uid, route, data)
}
```

### 3.4 玩家批量推送（Broadcast）

```go
// BroadcastToPlayers 向多个玩家推送消息
func BroadcastToPlayers(ctx context.Context, proxy *gate.Proxy, uids []int64, route int32, data interface{}) error {
    for _, uid := range uids {
        if err := proxy.Push(ctx, uid, route, data); err != nil {
            // 处理推送失败（玩家可能已离线）
            log.Warnf("推送失败 uid=%d, error=%v", uid, err)
        }
    }
    return nil
}
```

### 3.5 玩家离线处理

```go
// onDisconnect 玩家断开连接处理
func onDisconnect(ctx gate.Context) {
    session := ctx.Session()
    uid := session.Uid()
    if uid > 0 {
        log.Infof("玩家离线 uid=%d, cid=%d", uid, session.Cid())
        // TODO: 清理玩家相关数据
        // - 从房间中移除
        // - 通知好友
        // - 保存游戏状态
    }
}
```

### 3.6 玩家重连处理

```go
// reconnectHandler 重连处理器
func reconnectHandler(ctx gate.Context) {
    req := &struct {
        UID   int64  `json:"uid"`
        Token string `json:"token"`
    }{}

    if err := ctx.Parse(req); err != nil {
        return
    }

    // 验证 token 有效性

    // 重新绑定 UID，框架会自动处理之前的连接
    if err := ctx.Session().Bind(req.UID); err != nil {
        log.Errorf("重连绑定失败 uid=%d, error=%v", req.UID, err)
        return
    }

    log.Infof("玩家重连成功 uid=%d", req.UID)
}
```

### 3.7 Session 数据存储（Redis）

```go
import (
    "github.com/dobyte/due/v2/session/redis"
)

func setupSession() {
    store := redis.NewStore(
        redis.WithAddr("127.0.0.1:6379"),
        redis.WithPassword(""),
        redis.WithDB(0),
        redis.WithIdleTimeout(300), // 5 分钟空闲超时
    )
    session.SetStore(store)
}
```

---

## 4. 消息路由

Gate 自动处理消息路由到 Node，无需手动配置 Match 函数：

```
Client → Gate(自动路由) → Node(Proxy.Router 处理)
```

---

## 5. 配置选项

### 5.1 Gate 组件配置

```go
component := gate.NewGate(
    gate.WithID("gate-001"),        // 服务 ID
    gate.WithName("gate"),          // 服务名称
    gate.WithServer(server),        // 网络服务器
    gate.WithLocator(locator),      // 定位器
    gate.WithRegistry(registry),    // 注册中心
)
```

### 5.2 WebSocket 服务器配置

```go
server := ws.NewServer(
    ws.WithPort(8800),              // 端口
    ws.WithMaxConnNum(10000),       // 最大连接数
    ws.WithMsgSize(4096),           // 消息大小限制
    ws.WithHeartbeatInterval(30),   // 心跳间隔（秒）
)
```

### 5.3 定位器配置

```go
locator := redis.NewLocator(
    redis.WithAddr("127.0.0.1:6379"),
    redis.WithPassword("password"),
    redis.WithDB(0),
)
```

### 5.4 注册中心配置

```go
registry := etcd.NewRegistry(
    etcd.WithAddr("127.0.0.1:2379"),
    etcd.WithID("gate-001"),
    etcd.WithName("gate"),
)
```

---

## 6. 最佳实践

### ✅ 推荐做法

- 使用 Container 统一管理组件
- 配置服务注册实现服务发现
- 配置定位器用于消息路由
- 合理设置消息大小限制
- 配置心跳检测机制

### ❌ 避免做法

- 在 Gate 层执行业务逻辑
- 不使用 Container 直接调用 Serve()
- 忽略配置服务注册和定位器
- 不限制消息大小
