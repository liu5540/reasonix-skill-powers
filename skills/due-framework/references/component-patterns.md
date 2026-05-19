# due 组件使用指南

本文档介绍 `github.com/dobyte/due/v2` 框架提供的核心组件使用方法。

---

## 1. 日志组件（log/v2）

due 支持多种日志驱动：Console、File、Aliyun、Tencent。

### 1.1 Console 日志

```go
import "github.com/dobyte/due/v2/log"

// 设置日志级别
log.SetLevel(log.DebugLevel)

// 输出日志
log.Debug("调试信息")
log.Info("用户登录", "uid", 12345)
log.Warn("警告信息")
log.Error("错误信息", "error", err)
```

### 1.2 File 日志

```go
import (
    "github.com/dobyte/due/v2/log"
    "github.com/dobyte/due/log/file/v2"
)

// 创建 File 驱动
driver := file.NewDriver(
    file.WithDir("./logs"),
    file.WithFilename("game.log"),
    file.WithLevel(log.InfoLevel),
    file.WithMaxSize(100 * 1024 * 1024),  // 100MB
    file.WithMaxBackups(5),
    file.WithCompress(true),
)

// 设置日志
log.SetDriver(driver)
```

### 1.3 Aliyun 日志

```go
import (
    "github.com/dobyte/due/v2/log"
    "github.com/dobyte/due/log/aliyun/v2"
)

driver := aliyun.NewDriver(
    aliyun.WithEndpoint("cn-hangzhou.log.aliyuncs.com"),
    aliyun.WithAccessKeyID("your-access-key-id"),
    aliyun.WithAccessKeySecret("your-access-key-secret"),
    aliyun.WithProject("your-project"),
    aliyun.WithLogstore("your-logstore"),
)

log.SetDriver(driver)
```

### 1.4 Tencent 日志

```go
import (
    "github.com/dobyte/due/v2/log"
    "github.com/dobyte/due/log/tencent/v2"
)

driver := tencent.NewDriver(
    tencent.WithRegion("ap-guangzhou"),
    tencent.WithSecretID("your-secret-id"),
    tencent.WithSecretKey("your-secret-key"),
    tencent.WithTopicID("your-topic-id"),
)

log.SetDriver(driver)
```

### 1.5 结构化日志

```go
log.WithFields(log.Fields{
    "uid":   12345,
    "route": 1,
    "cost":  time.Millisecond * 100,
}).Info("用户请求")
```

---

## 2. 配置组件（config）

due 支持多种配置中心：Consul、Etcd、Nacos，支持 JSON/YAML/TOML/XML 格式。

### 2.1 Consul 配置

```go
import (
    "github.com/dobyte/due/config/consul/v2"
    "github.com/dobyte/due/v2/config"
)

source := consul.NewSource(
    consul.WithAddr("127.0.0.1:8500"),
    consul.WithPath("config/game/dev"),
)

cfg := config.NewConfig(config.WithSource(source))
if err := cfg.Load(); err != nil {
    panic(err)
}
```

### 2.2 Etcd 配置

```go
import (
    "github.com/dobyte/due/config/etcd/v2"
    "github.com/dobyte/due/v2/config"
)

source := etcd.NewSource(
    etcd.WithAddr("127.0.0.1:2379"),
    etcd.WithPath("config/game/dev"),
)

cfg := config.NewConfig(config.WithSource(source))
if err := cfg.Load(); err != nil {
    panic(err)
}
```

### 2.3 Nacos 配置

```go
import (
    "github.com/dobyte/due/config/nacos/v2"
    "github.com/dobyte/due/v2/config"
)

source := nacos.NewSource(
    nacos.WithAddr("127.0.0.1:8848"),
    nacos.WithNamespace("public"),
    nacos.WithGroup("DEFAULT_GROUP"),
    nacos.WithDataID("game-config"),
)

cfg := config.NewConfig(config.WithSource(source))
if err := cfg.Load(); err != nil {
    panic(err)
}
```

---

## 3. 缓存组件（cache）

### 3.1 Redis 缓存

```go
import (
    "github.com/dobyte/due/cache/redis/v2"
    "github.com/dobyte/due/v2/cache"
)

// 创建 Redis 缓存
c, err := redis.NewCache(
    redis.WithAddr("127.0.0.1:6379"),
    redis.WithPassword(""),
    redis.WithDB(0),
)
if err != nil {
    panic(err)
}

// 设置缓存
c.Set(ctx, "key", "value", time.Hour)

// 获取缓存
val, err := c.Get(ctx, "key")

// 删除缓存
c.Del(ctx, "key")
```

### 3.2 Memcache 缓存

```go
import (
    "github.com/dobyte/due/cache/memcache/v2"
    "github.com/dobyte/due/v2/cache"
)

c, err := memcache.NewCache(
    memcache.WithAddr("127.0.0.1:11211"),
)
if err != nil {
    panic(err)
}

c.Set(ctx, "key", "value", time.Hour)
```

---

## 4. 事件总线组件（eventbus）

### 4.1 Redis 事件总线

```go
import (
    "github.com/dobyte/due/eventbus/redis/v2"
    "github.com/dobyte/due/v2/eventbus"
)

// 创建事件总线
eb, err := redis.NewEventbus(
    redis.WithAddr("127.0.0.1:6379"),
    redis.WithPassword(""),
    redis.WithDB(0),
)
if err != nil {
    panic(err)
}

// 发布事件
err = eb.Publish(ctx, "topic", &Event{Type: "login", UID: 12345})

// 订阅事件
err = eb.Subscribe(ctx, "topic", func(event *Event) {
    log.Info("收到事件", "event", event)
})
```

### 4.2 NATS 事件总线

```go
import (
    "github.com/dobyte/due/eventbus/nats/v2"
    "github.com/dobyte/due/v2/eventbus"
)

eb, err := nats.NewEventbus(
    nats.WithAddr("nats://127.0.0.1:4222"),
)
if err != nil {
    panic(err)
}

err = eb.Publish(ctx, "topic", &Event{Type: "login", UID: 12345})
```

### 4.3 Kafka 事件总线

```go
import (
    "github.com/dobyte/due/eventbus/kafka/v2"
    "github.com/dobyte/due/v2/eventbus"
)

eb, err := kafka.NewEventbus(
    kafka.WithAddr("127.0.0.1:9092"),
    kafka.WithTopic("game-events"),
)
if err != nil {
    panic(err)
}
```

### 4.4 RabbitMQ 事件总线

```go
import (
    "github.com/dobyte/due/eventbus/rabbitmq/v2"
    "github.com/dobyte/due/v2/eventbus"
)

eb, err := rabbitmq.NewEventbus(
    rabbitmq.WithAddr("amqp://guest:guest@127.0.0.1:5672/"),
    rabbitmq.WithExchange("game-exchange"),
)
if err != nil {
    panic(err)
}
```

---

## 5. 注册中心组件（registry）

### 5.1 Consul 注册中心

```go
import "github.com/dobyte/due/registry/consul/v2"

registry := consul.NewRegistry(
    consul.WithAddr("127.0.0.1:8500"),
    consul.WithID("service-001"),
    consul.WithName("game-service"),
)
```

### 5.2 Etcd 注册中心

```go
import "github.com/dobyte/due/registry/etcd/v2"

registry := etcd.NewRegistry(
    etcd.WithAddr("127.0.0.1:2379"),
    etcd.WithID("service-001"),
    etcd.WithName("game-service"),
)
```

### 5.3 Nacos 注册中心

```go
import "github.com/dobyte/due/registry/nacos/v2"

registry := nacos.NewRegistry(
    nacos.WithAddr("127.0.0.1:8848"),
    nacos.WithNamespace("public"),
    nacos.WithServiceName("game-service"),
)
```

---

## 6. 分布式锁组件（lock）

### 6.1 Redis 分布式锁

```go
import (
    "github.com/dobyte/due/lock/redis/v2"
    "github.com/dobyte/due/v2/lock"
)

// 创建锁管理器
lm, err := redis.NewLockManager(
    redis.WithAddr("127.0.0.1:6379"),
    redis.WithPassword(""),
    redis.WithDB(5),
)
if err != nil {
    panic(err)
}

// 获取锁
l, err := lm.Acquire(ctx, "lock:key", time.Second*10)
if err != nil {
    panic(err)
}

// 释放锁
defer l.Release()

// 执行业务逻辑
```

### 6.2 Memcache 分布式锁

```go
import (
    "github.com/dobyte/due/lock/memcache/v2"
    "github.com/dobyte/due/v2/lock"
)

lm, err := memcache.NewLockManager(
    memcache.WithAddr("127.0.0.1:11211"),
)
if err != nil {
    panic(err)
}

l, err := lm.Acquire(ctx, "lock:key", time.Second*10)
if err != nil {
    panic(err)
}
defer l.Release()
```

---

## 7. 加密组件（crypto）

### 7.1 RSA 加密

```go
import "github.com/dobyte/due/crypto/rsa/v2"

// 生成密钥对
privateKey, publicKey, err := rsa.GenerateKeyPair(2048)
if err != nil {
    panic(err)
}

// 加密
ciphertext, err := rsa.Encrypt(publicKey, plaintext)

// 解密
plaintext, err := rsa.Decrypt(privateKey, ciphertext)
```

### 7.2 ECC 加密

```go
import "github.com/dobyte/due/crypto/ecc/v2"

// 生成密钥对
privateKey, publicKey, err := ecc.GenerateKeyPair(ecc.P256)
if err != nil {
    panic(err)
}

// 加密
ciphertext, err := ecc.Encrypt(publicKey, plaintext)

// 解密
plaintext, err := ecc.Decrypt(privateKey, ciphertext)
```

---

## 8. 传输器组件（transport）

### 8.1 RPCX 传输

```go
import "github.com/dobyte/due/transport/rpcx/v2"

transporter := rpcx.NewTransporter()

component := mesh.NewMesh(
    mesh.WithLocator(locator),
    mesh.WithRegistry(registry),
    mesh.WithTransporter(transporter),
)
```

### 8.2 gRPC 传输

```go
import "github.com/dobyte/due/transport/grpc/v2"

transporter := grpc.NewTransporter()

component := mesh.NewMesh(
    mesh.WithLocator(locator),
    mesh.WithRegistry(registry),
    mesh.WithTransporter(transporter),
)
```
