# due 架构设计模式

本文档介绍 `github.com/dobyte/due/v2` 框架的架构设计模式和核心概念。

---

## 1. 三层架构

due 采用 **Gate → Node → Mesh** 三层架构：

```
┌─────────────┐
│   Client    │
│ (TCP/KCP/WS)│
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│      Gate       │  ← 网关层：连接管理、消息路由
│  (无状态/多实例) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│      Node       │  ← 节点层：核心逻辑、Actor 模型
│   (有状态/单例)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│      Mesh       │  ← 微服务层：无状态业务逻辑
│  (无状态/多实例) │
└─────────────────┘
```

### 1.1 Gate（网关服）

**职责**：
- 管理客户端连接（TCP / KCP / WebSocket）
- 消息编解码和路由
- 会话管理（Session）
- 负载均衡到 Node

**特性**：
- 无状态设计，可水平扩展
- 支持多种协议
- 心跳检测和断线重连

**示例配置**：
```go
package main

import (
    "github.com/dobyte/due/network/ws/v2"
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/gate"
)

func main() {
    container := due.NewContainer()
    server := ws.NewServer(ws.WithPort(8800))
    locator := redis.NewLocator(redis.WithAddr("127.0.0.1:6379"))
    registry := etcd.NewRegistry(etcd.WithAddr("127.0.0.1:2379"))
    component := gate.NewGate(
        gate.WithID("gate-001"),
        gate.WithName("gate"),
        gate.WithServer(server),
        gate.WithLocator(locator),
        gate.WithRegistry(registry),
    )
    container.Add(component)
    container.Serve()
}
```

### 1.2 Node（节点服）

**职责**：
- 核心游戏逻辑处理
- Actor 状态管理
- 数据持久化
- 与其他服务通信

**特性**：
- 有状态设计，通常单实例
- Actor 模型隔离状态
- 消息队列保证顺序

**示例配置**：
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
    proxy.Router().AddRouteHandler(routeID, isSync, handlerFunc)
}
```

### 1.3 Mesh（微服务）

**职责**：
- 无状态业务逻辑
- 跨 Node 共享功能
- 第三方服务集成
- RPC 接口提供

**特性**：
- 无状态设计，可水平扩展
- RPCX / gRPC 接口
- 服务发现集成

---

## 2. 服务发现

due 支持多种服务发现后端，模块路径为 `/v2`：

### 2.1 etcd

```go
import "github.com/dobyte/due/registry/etcd/v2"

reg := etcd.NewRegistry(
    etcd.WithAddr("127.0.0.1:2379"),
    etcd.WithID("gate-001"),
    etcd.WithName("gate"),
)
```

### 2.2 Consul

```go
import "github.com/dobyte/due/registry/consul/v2"

reg := consul.NewRegistry(
    consul.WithAddr("127.0.0.1:8500"),
    consul.WithID("gate-001"),
    consul.WithName("gate"),
)
```

### 2.3 Nacos

```go
import "github.com/dobyte/due/registry/nacos/v2"

reg := nacos.NewRegistry(
    nacos.WithAddr("127.0.0.1:8848"),
    nacos.WithNamespace("public"),
)
```

---

## 3. 通信机制

### 3.1 Gate ↔ Node

通过 RPC 通信，支持 gRPC 和 RPCX：

```go
// RPCX 传输（推荐）
import "github.com/dobyte/due/transport/rpcx/v2"

trans := rpcx.NewTransporter()

component := node.NewNode(
    node.WithLocator(locator),
    node.WithRegistry(registry),
    node.WithTransporter(trans),
)
```

### 3.2 Node ↔ Mesh

支持多种通信方式：
- **RPCX**（默认推荐）
- **gRPC**（同步调用）
- **EventBus**（异步事件）
- **消息队列**（解耦处理）

---

## 4. Actor 模型

due 使用 Actor 模型处理有状态游戏逻辑：

```
┌─────────────────────────────────────┐
│              Node                   │
├─────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────┐ │
│  │ Actor A │  │ Actor B │  │ ... │ │
│  │ (Room)  │  │ (Player)│  │     │ │
│  └────┬────┘  └────┬────┘  └─────┘ │
│       │           │                │
│  ┌────┴───────────┴────┐           │
│  │   Message Queue     │           │
│  └─────────────────────┘           │
└─────────────────────────────────────┘
```

### 4.1 Actor 特性

- **隔离性**：每个 Actor 独立状态
- **顺序性**：消息顺序处理
- **并发性**：多 Actor 并行执行
- **位置透明**：可跨 Node 寻址

### 4.2 Actor 生命周期

在 due 中，Actor 逻辑通过路由处理器实现：

```go
import (
    "github.com/dobyte/due/v2/cluster/node"
)

type PlayerActor struct {
    uid   int64
    name  string
    level int
}

// 路由处理器函数
func playerHandler(ctx node.Context) {
    req := &Request{}
    res := &Response{}

    defer func() {
        if err := ctx.Response(res); err != nil {
            // 处理响应错误
        }
    }()

    if err := ctx.Parse(req); err != nil {
        res.Code = code.InternalError.Code()
        return
    }

    // 处理消息逻辑
    switch ctx.Route() {
    case LoginRoute:
        handleLogin(ctx, req, res)
    case MoveRoute:
        handleMove(ctx, req, res)
    }
}
```

---

## 5. 数据流

### 5.1 请求流程

```
Client → Gate(协议解析) → Node(Actor 路由) → Actor(业务逻辑)
                                      ↓
                                   (数据持久化)
```

### 5.2 响应流程

```
Actor(响应数据) → Gate(协议封装) → Client
```

---

## 6. 部署架构

### 6.1 开发环境

```
┌──────────────┐
│ docker-compose│
├──────────────┤
│  Gate :8800  │
│  Node :9000  │
│  Redis :6379 │
│  etcd  :2379 │
└──────────────┘
```

### 6.2 生产环境

```
              ┌───────────┐
              │   SLB     │
              └─────┬─────┘
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   ┌────────┐ ┌────────┐ ┌────────┐
   │ Gate 1 │ │ Gate 2 │ │ Gate 3 │
   └────┬───┘ └────┬───┘ └────┬───┘
        │          │          │
        └──────────┼──────────┘
                   ▼
        ┌──────────────────┐
        │      etcd        │
        │   服务发现/配置   │
        └────────┬─────────┘
                 ▼
   ┌─────────────┼─────────────┐
   ▼             ▼             ▼
┌───────┐   ┌───────┐   ┌───────┐
│Node 1 │   │Node 2 │   │Node 3 │
└───────┘   └───────┘   └───────┘
```

---

## 7. 最佳实践

### ✅ 推荐做法

- Gate 无状态，可多实例
- Node 使用 Actor 管理状态
- 服务注册使用一致的后端
- 配置中心管理环境变量
- EventBus 处理跨服务通信

### ❌ 避免做法

- Gate 层执行业务逻辑
- 全局共享状态（使用 Actor）
- 硬编码服务地址
- 阻塞 Actor 消息循环
