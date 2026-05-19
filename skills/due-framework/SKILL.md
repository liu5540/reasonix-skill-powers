---
name: due-framework
description: |
  本项目基于 `github.com/dobyte/due/v2` 分布式游戏服务器框架（v2.5.x）。

  **使用场景：**
  - 编写或修改 Gate（网关）、Node（节点）、Mesh（微服务）相关代码
  - 处理 WebSocket 连接管理、消息路由、会话绑定
  - 实现 Actor 模型状态机、牌桌逻辑、游戏模块
  - 配置服务发现（etcd/Consul/Nacos）、事件总线（NATS/Redis）
  - 使用 due 内置组件（日志、缓存、配置中心、分布式锁、传输器）
  - 处理 Protobuf 消息序列化、自定义路由、心跳机制
  - 管理 Container 生命周期、组件组装与服务注册

  **必须调用本技能的情况：**
  任何涉及 due 框架 API（`due.NewContainer`、`gate.NewGate`、`node.NewNode`、`mesh.NewMesh`、`node.Context`、`proxy.Router()`、`AddRouteHandler`、`AddServiceProvider`、`GetServiceClient` 等）的开发、调试或重构任务。
---

# due 框架开发技能（v2）

> 本技能面向 AI 编码助手，提供 `github.com/dobyte/due/v2` 框架的完整开发指南。
> 与项目 `AGENTS.md`（项目特定约定）和 `docs/devlop/`（开发规范）互补。

---

## 1. 框架定位

due 是轻量级、高性能的分布式游戏服务器框架（Apache 2.0 许可证），核心特性：
- **Gate → Node → Mesh** 三层架构
- **Actor 模型**处理有状态游戏逻辑
- **多协议支持**（WebSocket / TCP / KCP）
- **服务发现**（etcd / Consul / Nacos）
- **事件总线**（NATS / Redis / Kafka / RabbitMQ）

本项目使用 due v2.5.5，模块路径统一为 `github.com/dobyte/due/v2` 及其子模块。

---

## 2. 知识索引

按需加载对应参考文档，不要一次性阅读全部内容：

| 文件 | 加载时机 | 内容 |
|------|----------|------|
| [references/architecture-patterns.md](references/architecture-patterns.md) | 设计服务拓扑、理解架构 | Gate/Node/Mesh 三层架构、服务发现、Actor 模型 |
| [references/gate-patterns.md](references/gate-patterns.md) | 开发网关服务 | WebSocket/TCP/KCP 接入、会话管理、消息路由、心跳 |
| [references/node-patterns.md](references/node-patterns.md) | 开发游戏逻辑节点 | Actor 模型、路由处理器、消息传递、状态管理 |
| [references/mesh-patterns.md](references/mesh-patterns.md) | 开发 RPC 微服务 | 无状态服务设计、服务提供者注册、RPC 调用 |
| [references/component-patterns.md](references/component-patterns.md) | 使用框架组件 | 日志、缓存、事件总线、配置中心、注册中心、分布式锁、传输器 |
| [references/protocol-patterns.md](references/protocol-patterns.md) | 定义通信协议 | 数据包格式、心跳包、序列化、路由配置 |

---

## 3. 核心原则

### ✅ 必须遵守

- **三层分离**：Gate 只管连接，Node 管有状态逻辑，Mesh 管无状态业务
- **Actor 无锁**：同一线程串行处理消息，Actor 内部不加业务锁
- **消息路由**：所有通信必须定义明确的路由号（route）
- **服务注册**：所有服务必须通过注册中心注册，禁止硬编码地址
- **配置外置**：使用配置中心（etcd/Consul/Nacos）管理环境配置
- **Session 隔离**：玩家状态绑定到 Session，禁止全局可变状态

### ❌ 严禁行为

- 在 Gate 层写业务逻辑（违反分层）
- Actor 内使用全局变量共享状态
- 硬编码服务 IP 端口（必须走服务发现）
- 在路由 handler 中跳过消息校验
- 阻塞 Actor 消息循环（长操作必须异步）
- 不处理连接断开事件（资源泄漏）

---

## 4. 常用模块路径

```
github.com/dobyte/due/v2                    # 主框架
github.com/dobyte/due/locate/redis/v2       # Redis 定位器
github.com/dobyte/due/network/ws/v2         # WebSocket
github.com/dobyte/due/network/tcp/v2        # TCP
github.com/dobyte/due/network/kcp/v2        # KCP
github.com/dobyte/due/registry/etcd/v2      # etcd 注册中心
github.com/dobyte/due/registry/consul/v2    # Consul 注册中心
github.com/dobyte/due/registry/nacos/v2     # Nacos 注册中心
github.com/dobyte/due/transport/rpcx/v2     # RPCX 传输器
github.com/dobyte/due/transport/grpc/v2     # gRPC 传输器
```

---

## 5. 快速示例

### Gate 服务

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
    server := ws.NewServer(ws.WithPort(8800))
    locator := redis.NewLocator(redis.WithAddr("127.0.0.1:6379"))
    registry := etcd.NewRegistry(etcd.WithAddr("127.0.0.1:2379"))
    component := gate.NewGate(
        gate.WithServer(server),
        gate.WithLocator(locator),
        gate.WithRegistry(registry),
    )
    container.Add(component)
    container.Serve()
}
```

### Node 服务

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

func initListen(proxy *node.Proxy) {
    proxy.Router().AddRouteHandler(routeID, isSync, handlerFunc)
}
```

### Mesh 服务

```go
package main

import (
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/transport/rpcx/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/mesh"
)

func main() {
    container := due.NewContainer()
    locator := redis.NewLocator()
    registry := etcd.NewRegistry()
    transporter := rpcx.NewTransporter()
    component := mesh.NewMesh(
        mesh.WithLocator(locator),
        mesh.WithRegistry(registry),
        mesh.WithTransporter(transporter),
    )
    component.AddServiceProvider("User", &rpcpb.UserDesc{}, &UserService{})
    container.Add(component)
    container.Serve()
}
```

---

## 6. 自检清单

处理 due 框架相关代码时，确认以下要点：

- [ ] 服务是否通过 `due.NewContainer()` 管理生命周期
- [ ] Gate/Node/Mesh 职责是否清晰分离
- [ ] Actor 内部是否无锁（单线程保证）
- [ ] 路由 handler 是否注册了延迟响应（`defer ctx.Response` 或 `ctx.Defer`）
- [ ] 消息是否经过 `ctx.Parse()` 解析和校验
- [ ] 服务间调用是否通过 `proxy.GetServiceClient()` 而非硬编码
- [ ] 配置是否外置到配置中心
- [ ] 日志是否结构化（`zap` 字段形式）
