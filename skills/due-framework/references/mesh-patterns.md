# due 微服务开发模式

本文档介绍 `github.com/dobyte/due/v2` 框架中 Mesh（微服务）的开发模式。

---

## 1. Mesh 概述

Mesh 服务是 due 架构中的无状态微服务层，负责：
- 无状态业务逻辑处理
- 跨 Node 共享功能
- 第三方服务集成
- RPC 接口提供（使用 RPCX / gRPC）

---

## 2. 与 Gate/Node 的区别

| 特性 | Gate | Node | Mesh |
|------|------|------|------|
| 状态 | 无状态 | 有状态 | 无状态 |
| 扩展 | 水平扩展 | 单实例 | 水平扩展 |
| 协议 | TCP/KCP/WS | Actor 消息 | RPCX/gRPC |
| 用途 | 连接管理 | 游戏逻辑 | 业务服务 |

---

## 3. 创建 Mesh 服务

### 3.1 基础 Mesh 服务

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
    locator := redis.NewLocator(redis.WithAddr("127.0.0.1:6379"))
    registry := etcd.NewRegistry(etcd.WithAddr("127.0.0.1:2379"))
    transporter := rpcx.NewTransporter()
    component := mesh.NewMesh(
        mesh.WithID("mesh-001"),
        mesh.WithName("mesh"),
        mesh.WithLocator(locator),
        mesh.WithRegistry(registry),
        mesh.WithTransporter(transporter),
    )
    component.AddServiceProvider("user.service", "UserService", &UserService{})
    container.Add(component)
    container.Serve()
}

// 用户服务实现
type UserService struct{}

func (s *UserService) GetUser(ctx context.Context, req *GetUserRequest, res *GetUserResponse) error {
    res.UID = req.UID
    res.Name = "Alice"
    return nil
}
```

### 3.2 完整 Mesh 配置

```go
component := mesh.NewMesh(
    mesh.WithID("mesh-001"),
    mesh.WithName("mesh"),
    mesh.WithLocator(locator),
    mesh.WithRegistry(registry),
    mesh.WithTransporter(transporter),
)
```

---

## 4. 服务间通信

### 4.1 Mesh 调用 Node

```go
import (
    "github.com/dobyte/due/v2/message"
)

// 发送消息到 Actor
func callPlayerActor(uid int64, route int64, data interface{}) error {
    return message.Send(uid, route, data)
}

// 广播消息
func broadcastToPlayers(uids []int64, route int64, data interface{}) {
    message.Broadcast(uids, route, data)
}
```

### 4.2 Mesh 调用 Mesh

```go
import (
    "github.com/dobyte/due/v2/proxy"
)

// 通过 Proxy 调用其他 Mesh 服务
func callOtherMesh(proxy *proxy.Proxy, serviceName string, method string, req interface{}) (interface{}, error) {
    res, err := proxy.Call(serviceName, method, req)
    if err != nil {
        return nil, err
    }
    return res, nil
}
```

---

## 5. 配置管理

```go
type MeshConfig struct {
    ID        string `json:"id"`
    Name      string `json:"name"`
    EtcdAddr  string `json:"etcd_addr"`
    RedisAddr string `json:"redis_addr"`
}

func loadConfig() *MeshConfig {
    return &MeshConfig{
        ID:        "mesh-001",
        Name:      "mesh",
        EtcdAddr:  "127.0.0.1:2379",
        RedisAddr: "127.0.0.1:6379",
    }
}

func main() {
    c := loadConfig()
    container := due.NewContainer()
    locator := redis.NewLocator(redis.WithAddr(c.RedisAddr))
    registry := etcd.NewRegistry(etcd.WithAddr(c.EtcdAddr))
    transporter := rpcx.NewTransporter()
    component := mesh.NewMesh(
        mesh.WithID(c.ID),
        mesh.WithName(c.Name),
        mesh.WithLocator(locator),
        mesh.WithRegistry(registry),
        mesh.WithTransporter(transporter),
    )
    container.Add(component)
    container.Serve()
}
```

---

## 6. 完整示例

### 6.1 用户服务 Mesh 实现

```go
package main

import (
    "context"
    "github.com/dobyte/due/locate/redis/v2"
    "github.com/dobyte/due/registry/etcd/v2"
    "github.com/dobyte/due/transport/rpcx/v2"
    "github.com/dobyte/due/v2"
    "github.com/dobyte/due/v2/cluster/mesh"
)

func main() {
    container := due.NewContainer()
    locator := redis.NewLocator(redis.WithAddr("127.0.0.1:6379"))
    registry := etcd.NewRegistry(etcd.WithAddr("127.0.0.1:2379"))
    transporter := rpcx.NewTransporter()
    component := mesh.NewMesh(
        mesh.WithID("mesh-user-001"),
        mesh.WithName("mesh-user"),
        mesh.WithLocator(locator),
        mesh.WithRegistry(registry),
        mesh.WithTransporter(transporter),
    )
    userService := &UserService{}
    component.AddServiceProvider("user.service", "UserService", userService)
    container.Add(component)
    container.Serve()
}

// 用户服务
type UserService struct{}

type GetUserRequest struct {
    UID int64 `json:"uid"`
}

type GetUserResponse struct {
    Code int    `json:"code"`
    UID  int64  `json:"uid"`
    Name string `json:"name"`
}

func (s *UserService) GetUser(ctx context.Context, req *GetUserRequest, res *GetUserResponse) error {
    res.Code = 0
    res.UID = req.UID
    res.Name = "Alice"
    return nil
}

type LoginRequest struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

type LoginResponse struct {
    Code  int    `json:"code"`
    Token string `json:"token"`
    UID   int64  `json:"uid"`
}

func (s *UserService) Login(ctx context.Context, req *LoginRequest, res *LoginResponse) error {
    if req.Username == "admin" && req.Password == "123456" {
        res.Code = 0
        res.Token = "generated_token"
        res.UID = 1
    } else {
        res.Code = 1
    }
    return nil
}
```

---

## 7. 变化说明

**重要**：due v2 调整了组件使用方式：

1. **使用 Container 统一管理**：所有组件通过 `due.NewContainer()` 管理
2. **Mesh 作为组件**：使用 `mesh.NewMesh()` 创建组件
3. **服务提供者注册**：使用 `AddServiceProvider()` 注册服务
4. **RPCX 传输**：默认使用 RPCX 作为传输协议

---

## 8. 最佳实践

### ✅ 推荐做法

- 使用 Container 统一管理组件
- 使用服务注册实现服务发现
- 实现统一的错误处理
- 对输入进行严格验证
- 使用服务发现进行服务间调用

### ❌ 避免做法

- 在 Mesh 中存储有状态数据
- 忽略错误处理
- 不使用 Container 直接调用 Serve()
- 硬编码服务地址
