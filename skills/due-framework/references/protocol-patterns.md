# due 通信协议模式

本文档介绍 `github.com/dobyte/due/v2` 框架中的通信协议格式和序列化模式。

---

## 1. 数据包格式

### 1.1 默认格式

due 默认使用以下数据包格式：

```
┌──────────┬───────────┬──────────┬───────────┬───────────┐
│   size   │  header   │   route  │    seq    │  message  │
│ 2 bytes  │ 1 byte    │ 2 bytes  │ 2 bytes   │ N bytes   │
└──────────┴───────────┴──────────┴───────────┴───────────┘
```

#### 字段说明

| 字段 | 大小 | 说明 |
|------|------|------|
| size | 2 bytes | 数据包总长度（不包含 size 本身） |
| header | 1 byte | 消息头标识（用于区分消息类型） |
| route | 2 bytes | 消息路由/类型 ID |
| seq | 2 bytes | 序列号，用于请求-响应匹配 |
| message | N bytes | 消息体（序列化后的数据） |

### 1.2 心跳包格式

```
┌──────────┬───────────┬───────────┬────────────────┐
│   size   │  header   │  extcode  │  heartbeat_time│
│ 2 bytes  │ 1 byte    │ 1 byte    │ 4 bytes       │
└──────────┴───────────┴───────────┴────────────────┘
```

#### 字段说明

| 字段 | 大小 | 说明 |
|------|------|------|
| size | 2 bytes | 心跳包总长度 |
| header | 1 byte | 心跳标识（固定值） |
| extcode | 1 byte | 扩展码 |
| heartbeat_time | 4 bytes | 心跳时间戳 |

---

## 2. 序列化器

### 2.1 JSON 序列化器

```go
import "github.com/dobyte/due/core/serializer/json"

serializer := json.NewSerializer()

// 序列化
data, err := serializer.Marshal(message)

// 反序列化
err := serializer.Unmarshal(data, &message)
```

### 2.2 Protobuf 序列化器

```go
import "github.com/dobyte/due/core/serializer/protobuf"

serializer := protobuf.NewSerializer()

// 序列化
data, err := serializer.Marshal(protoMessage)

// 反序列化
err := serializer.Unmarshal(data, &protoMessage)
```

### 2.3 自定义序列化器

实现 Serializer 接口：

```go
type Serializer interface {
    Marshal(v interface{}) ([]byte, error)
    Unmarshal(data []byte, v interface{}) error
}

// 自定义实现
type CustomSerializer struct{}

func (s *CustomSerializer) Marshal(v interface{}) ([]byte, error) {
    // 自定义序列化逻辑
    return data, nil
}

func (s *CustomSerializer) Unmarshal(data []byte, v interface{}) error {
    // 自定义反序列化逻辑
    return nil
}
```

---

## 3. 协议配置

### 3.1 自定义数据包格式

```go
import (
    "github.com/dobyte/due/v2/packet"
    "github.com/dobyte/due/core/serializer/json"
)

// 创建自定义数据包
p := packet.NewPacket(
    packet.WithSizeBytes(2),      // size 字段 2 字节
    packet.WithRouteBytes(2),     // route 字段 2 字节
    packet.WithSeqBytes(2),       // seq 字段 2 字节
    packet.WithSerializer(json.NewSerializer()),
)
```

### 3.2 自定义路由配置

```go
import "github.com/dobyte/due/v2/route"

// 注册路由
router := route.NewRouter()
router.AddRoute(1, "login")
router.AddRoute(2, "move")
router.AddRoute(3, "chat")

// 获取路由名称
name := router.GetRouteName(1) // "login"
```

---

## 4. 消息处理流程

### 4.1 发送消息

```go
import (
    "github.com/dobyte/due/v2/message"
    "github.com/dobyte/due/core/serializer/json"
)

// 创建消息
msg := &message.Message{
    Route: 1,
    Seq:   1001,
    Data:  data,
}

// 序列化
serializer := json.NewSerializer()
data, err := serializer.Marshal(msg)
```

### 4.2 接收消息

```go
// 解析数据包
p := packet.NewPacket()
msg, err := p.Unpack(data)
if err != nil {
    log.Error("解析消息失败", "error", err)
    return
}

// 根据路由处理
switch msg.Route {
case LoginRoute:
    handleLogin(msg)
case MoveRoute:
    handleMove(msg)
}
```

---

## 5. 心跳机制

### 5.1 服务端心跳配置

```go
import (
    "github.com/dobyte/due/network/ws/v2"
    "time"
)

server := ws.NewServer(
    ws.WithHeartbeatInterval(30),           // 30 秒心跳间隔
    ws.WithHeartbeatIdleTime(time.Minute),  // 1 分钟空闲超时
)
```

### 5.2 客户端心跳实现

```javascript
// 客户端 JavaScript 示例
const heartbeatInterval = 25000; // 25 秒发送一次

setInterval(() => {
    const heartbeat = new Uint8Array([
        0x00, 0x07,  // size: 7 bytes
        0x01,        // header: heartbeat
        0x00,        // extcode
        0x00, 0x00, 0x00, 0x00, // timestamp
    ]);
    ws.send(heartbeat);
}, heartbeatInterval);
```

---

## 6. 协议最佳实践

### ✅ 推荐做法

- 使用 Protobuf 序列化（性能更好，消息更小）
- 为每个路由定义常量，避免魔法数字
- 使用序列号（seq）匹配请求和响应
- 合理设置消息大小限制（防止内存溢出）
- 心跳间隔小于空闲超时时间

### ❌ 避免做法

- 不校验消息大小直接解析（安全风险）
- 忽略序列号导致请求响应不匹配
- 心跳间隔等于或大于超时时间
- 在消息体中传输敏感信息（应加密）
