# due 节点开发模式

本文档介绍 `github.com/dobyte/due/v2` 框架中 Node 服务的开发模式，重点介绍 Actor 模型的使用。

---

## 1. Node 概述

Node 服务是游戏服务器的核心，负责：
- 使用 Actor 模型处理有状态游戏逻辑
- 接收 Gate 转发的客户端消息
- 数据持久化
- 与其他服务通信

---

## 2. 完整示例

```go
package main

import (
    "fmt"
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/node"
    "github.com/dobyte/due/v2/log"
)

const greetRoute = 1

func main() {
    container := due.NewContainer()
    locator := redis.NewLocator(redis.WithAddr("127.0.0.1:6379"))
    registry := etcd.NewRegistry(etcd.WithAddr("127.0.0.1:2379"))
    component := node.NewNode(
        node.WithID("node-001"),
        node.WithName("node"),
        node.WithLocator(locator),
        node.WithRegistry(registry),
    )
    initListen(component.Proxy())
    container.Add(component)
    container.Serve()
}

func initListen(proxy *node.Proxy) {
    proxy.Router().AddRouteHandler(greetRoute, false, greetHandler)
}

type greetReq struct {
    Message string `json:"message"`
}

type greetRes struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
}

func greetHandler(ctx node.Context) {
    req := &greetReq{}
    res := &greetRes{}

    defer func() {
        if err := ctx.Response(res); err != nil {
            log.Errorf("response message failed: %v", err)
        }
    }()

    if err := ctx.Parse(req); err != nil {
        log.Errorf("parse request message failed: %v", err)
        res.Code = code.InternalError.Code()
        return
    }

    log.Info(req.Message)
    res.Code = code.OK.Code()
    res.Message = fmt.Sprintf("I'm server, current time: %s", time.Now().Format(time.DateTime))
}
```

---

## 3. Actor 模型基础

### 3.1 什么是 Actor

Actor 是并发编程的基本单元，具有以下特性：
- **独立状态**：每个 Actor 拥有独立的状态
- **消息驱动**：通过消息进行通信
- **顺序处理**：消息按顺序处理
- **位置透明**：可以在任何 Node 上运行

### 3.2 Actor 生命周期

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Init   │ ──▶ │ Running │ ──▶ │Stopping │ ──▶ │ Destroy │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
```

---

## 4. 路由处理器

在 due 中，Actor 逻辑通过路由处理器实现。

### 4.1 基础路由处理器

```go
const (
    LoginRoute = 1
    MoveRoute  = 2
    ChatRoute  = 3
)

func initListen(proxy *node.Proxy) {
    proxy.Router().AddRouteHandler(LoginRoute, false, loginHandler)
    proxy.Router().AddRouteHandler(MoveRoute, false, moveHandler)
    proxy.Router().AddRouteHandler(ChatRoute, false, chatHandler)
}

func loginHandler(ctx node.Context) {
    req := &LoginRequest{}
    res := &LoginResponse{}

    defer func() {
        ctx.Response(res)
    }()

    if err := ctx.Parse(req); err != nil {
        res.Code = code.InternalError.Code()
        return
    }

    // 处理登录逻辑
    res.Code = code.OK.Code()
    res.UID = req.UID
}

func moveHandler(ctx node.Context) {
    req := &MoveRequest{}
    res := &MoveResponse{}

    defer func() {
        ctx.Response(res)
    }()

    if err := ctx.Parse(req); err != nil {
        res.Code = code.InternalError.Code()
        return
    }

    // 处理移动逻辑
}

func chatHandler(ctx node.Context) {
    req := &ChatRequest{}
    res := &ChatResponse{}

    defer func() {
        ctx.Response(res)
    }()

    if err := ctx.Parse(req); err != nil {
        res.Code = code.InternalError.Code()
        return
    }

    // 处理聊天逻辑
}
```

### 4.2 同步与异步处理

```go
// 异步处理（推荐）- isSync = false
proxy.Router().AddRouteHandler(routeID, false, handler)

// 同步处理 - isSync = true
// 同步模式下，消息按顺序处理，适用于需要严格顺序的场景
proxy.Router().AddRouteHandler(routeID, true, handler)
```

---

## 5. 创建 Node 服务

### 5.1 基础 Node

```go
package main

import (
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/node"
)

func main() {
    container := due.NewContainer()
    locator := redis.NewLocator()
    registry := etcd.NewRegistry()
    component := node.NewNode(
        node.WithLocator(locator),
        node.WithRegistry(registry),
    )
    initListen(component.Proxy())
    container.Add(component)
    container.Serve()
}
```

### 5.2 完整 Node 配置

```go
component := node.NewNode(
    node.WithID("node-001"),
    node.WithName("node"),
    node.WithLocator(locator),
    node.WithRegistry(registry),
    node.WithWorkerSize(32),         // Worker 数量
)
```

---

## 6. 消息处理

### 6.1 消息结构

```go
// 请求消息
type LoginRequest struct {
    UID      int64  `json:"uid"`
    Token    string `json:"token"`
}

// 响应消息
type LoginResponse struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
    UID     int64  `json:"uid"`
}
```

### 6.2 Context 接口

```go
// node.Context 提供的方法
ctx.Route()      // 获取路由 ID
ctx.Session()    // 获取会话
ctx.Parse(req)   // 解析请求数据
ctx.Response(res)// 发送响应
ctx.Uid()        // 获取用户 ID
ctx.Cid()        // 获取连接 ID
```

---

## 7. Actor 间通信

### 7.1 发送消息

```go
import "github.com/dobyte/due/v2/message"

// 发送消息给特定用户
func sendMessage(uid int64, route int64, data interface{}) {
    message.Send(uid, route, data)
}

// 广播消息给多个用户
func broadcastMessage(uids []int64, route int64, data interface{}) {
    message.Broadcast(uids, route, data)
}
```

### 7.2 推送消息

```go
// 通过 Proxy 推送消息
func pushMessage(proxy *node.Proxy, uid int64, route int64, data interface{}) {
    proxy.Push(uid, route, data)
}
```

---

## 8. 数据持久化

### 8.1 使用 Redis 缓存

```go
import (
    "github.com/dobyte/due/redis/v2"
)

type PlayerData struct {
    UID   int64  `json:"uid"`
    Name  string `json:"name"`
    Level int    `json:"level"`
}

func savePlayerData(client *redis.Client, data *PlayerData) error {
    key := fmt.Sprintf("player:%d", data.UID)
    return client.Set(key, data).Err()
}

func loadPlayerData(client *redis.Client, uid int64) (*PlayerData, error) {
    key := fmt.Sprintf("player:%d", uid)
    data := &PlayerData{}
    err := client.Get(key).Scan(data)
    return data, err
}
```

---

## 9. 最佳实践

### ✅ 推荐做法

- 使用路由处理器处理消息
- 使用 `ctx.Parse()` 解析请求
- 使用 `ctx.Response()` 发送响应
- 使用 Container 统一管理组件
- 实现错误码规范

### ❌ 避免做法

- 在 Node 层直接写业务逻辑（应在 Actor/Handler 中）
- 忽略错误处理
- 不使用 Container 直接调用 Serve()
- 硬编码路由 ID（使用常量定义）
