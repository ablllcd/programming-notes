# MQTT理论
## 协议简介

MQTT（Message Queuing Telemetry Transport）是一种轻量级的发布/订阅消息传输协议，专为低带宽、不可靠网络环境设计。它通常用于物联网设备之间的通信。

### MQTT的特点
- **轻量级**：协议设计简单，适合资源受限的设备。
- **发布/订阅模式**：支持多对多的消息传递。
- **低带宽**：优化了网络使用，适合低速网络。
- **可靠性**：支持三种消息质量服务（QoS）级别。
- **持久化**：支持离线消息存储和传递。

### MQTT的核心概念
1. **Broker（代理服务器）**：负责接收和分发消息。
2. **Client（客户端）**：发布消息或订阅主题的设备。
3. **Topic（主题）**：消息的分类标识，用于订阅和发布。
4. **QoS（服务质量）**：
    - **QoS 0**：最多一次传递，不保证消息到达。
    - **QoS 1**：至少一次传递，可能重复。
    - **QoS 2**：仅一次传递，确保消息不重复。

### MQTT的工作流程
1. 客户端连接到 Broker。
2. 客户端订阅一个或多个主题。
3. 客户端发布消息到某个主题。
4. Broker 将消息分发给订阅该主题的客户端。

### MQTT的应用场景
- 智能家居：设备状态监控与控制。
- 工业物联网：传感器数据采集与分析。
- 移动应用：实时消息推送。
- 远程监控：设备故障报警与诊断。

## MQTT消息传输
在 MQTT（Message Queuing Telemetry Transport）协议中，消息的生产和消耗通过 **发布（Publish）** 和 **订阅（Subscribe）** 的机制来实现。以下是具体的生产和消耗过程：

---

### 1. 消息的生产（发布）
消息的生产由 **发布者（Publisher）** 完成。发布者将消息发送到一个特定的 **主题（Topic）**。主题是消息的分类标识，类似于一个路径或标签。

#### 发布过程：
1. **连接到 MQTT Broker**：发布者首先与 MQTT Broker 建立连接。
2. **指定主题**：发布者选择一个主题（如 `sensor/temperature`）。
3. **发送消息**：发布者将消息内容（如温度数据 `25°C`）发送到指定的主题。
4. **Broker 分发消息**：Broker 接收到消息后，会根据订阅关系将消息分发给订阅该主题的客户端。

---

### 2. 消息的消耗（订阅）
消息的消耗由 **订阅者（Subscriber）** 完成。订阅者通过订阅一个或多个主题来接收消息。

#### 订阅过程：
1. **连接到 MQTT Broker**：订阅者与 MQTT Broker 建立连接。
2. **订阅主题**：订阅者向 Broker 发送订阅请求，指定感兴趣的主题（如 `sensor/temperature`）。
3. **接收消息**：当 Broker 接收到与订阅主题匹配的消息时，会将消息推送给订阅者。

---

### 3. Broker 的作用
MQTT Broker 是消息的中间人，负责：
- **存储和转发消息**：接收发布者的消息，并将消息分发给订阅者。
- **管理订阅关系**：记录哪些客户端订阅了哪些主题。
- **支持 QoS（服务质量）**：根据 QoS 等级（0、1、2）确保消息的可靠传递。

---

## QoS 服务质量

### QoS 0 - 最多一次传递（At most once）

* 发送即忘记（Fire and forget）
* 不保证消息到达
* 没有确认机制
* 最快的传输方式

### QoS 1 - 至少一次传递（At least once）

#### 特点：
- 保证消息至少到达一次。
- 可能出现消息重复。
- 需要 `PUBACK` 确认。
- 发送方会重试直到收到确认。

#### 流程：
1. 发送方发送 `PUBLISH` 消息。
2. 接收方回复 `PUBACK` 确认。
3. 如果超时未收到 `PUBACK`，发送方会重发消息。


### QoS 2 - 仅一次传递（Exactly once）

#### 特点：
- 保证消息仅传递一次。
- 最高可靠性级别。
- 使用四次握手确认机制。
- 传输开销最大。

#### 流程：
1. 发送方发送 `PUBLISH` 消息。
2. 接收方回复 `PUBREC`。
3. 发送方发送 `PUBREL`。
4. 接收方回复 `PUBCOMP`。

---

### QoS 控制的两个独立阶段

 1. 发布端到 Broker 的 QoS
    - 由发布消息时指定的 QoS 控制。
    - 决定发布端如何向 Broker 传递消息。

2. Broker 到订阅端的 QoS
    - 由订阅时指定的 QoS 控制。
    - 决定 Broker 如何向订阅端传递消息。

3. 实际生效的 QoS 规则
    - 最终生效的 QoS 等级为：`min(发布 QoS, 订阅 QoS)`。
    - 即消息传递过程中，实际使用的 QoS 是发布端和订阅端指定的 QoS 中较低的等级。

### 如果先发布消息，然后才有人订阅会怎样？
在 MQTT 中，如果消息发布时没有订阅者，消息的处理情况取决于以下因素：

#### 1. **QoS（服务质量）**
- **QoS 0（最多一次传递）**：消息不会被存储，Broker 只会尝试将消息立即发送给当前在线的订阅者。如果没有订阅者，消息会被丢弃。
- **QoS 1（至少一次传递）**：Broker 会存储消息，确保至少发送一次给订阅者。但如果没有订阅者，消息仍然会被丢弃，除非启用了持久化设置。
- **QoS 2（仅一次传递）**：Broker 会存储消息，并确保消息仅被发送一次给订阅者。如果没有订阅者，消息同样会被丢弃，除非启用了持久化设置。

#### 2. **持久化订阅（Retained Messages）**
如果主题启用了 **Retained Messages**（保留消息），Broker 会保存最新的消息，即使没有订阅者。当新的订阅者订阅该主题时，Broker 会立即将保留的消息发送给订阅者。

#### 示例：
发布消息时使用 `-r` 参数（保留消息）：
```bash
mosquitto_pub -h localhost -p 1883 -t test/topic -m "Hello MQTT" -r
```
订阅者稍后订阅该主题时，仍然可以收到之前发布的消息：
```bash
mosquitto_sub -h localhost -p 1883 -t test/topic
```


# MQTT软件及使用
## Mosquitto MQTT Broker

Mosquitto 是一个开源的 MQTT 代理服务器，广泛用于物联网 (IoT) 应用中。它支持 MQTT 协议的所有版本，并且可以在多种操作系统上运行。

### Linux安装 Mosquitto

```bash 
sudo apt update
sudo apt install mosquitto mosquitto-clients
```

### 启动 Mosquitto 服务

```bash 
sudo systemctl start mosquitto
```

### 检查 Mosquitto 服务状态

```bash
sudo systemctl status mosquitto
```

### 查看 MQTT Broker 消息

可以使用 Mosquitto 客户端工具 `mosquitto_sub` 和 `mosquitto_pub` 来查看和测试消息传递。

#### 订阅主题以查看消息

使用 `mosquitto_sub` 命令订阅一个主题，实时查看该主题的消息：

```bash
mosquitto_sub -h <broker_address> -p <port> -t <topic>
```

示例：

```bash
mosquitto_sub -h localhost -p 1883 -t test/topic
```

#### 发布消息到主题

使用 `mosquitto_pub` 命令向某个主题发布消息：

```bash
mosquitto_pub -h <broker_address> -p <port> -t <topic> -m "<message>"
```

示例：

```bash
mosquitto_pub -h localhost -p 1883 -t test/topic -m "Hello MQTT"
```

#### 验证消息传递

1. 在一个终端窗口运行 `mosquitto_sub` 订阅主题。
2. 在另一个终端窗口运行 `mosquitto_pub` 发布消息。
3. 在订阅窗口中可以看到发布的消息。

#### 使用日志查看消息

如果启用了 Mosquitto 的日志功能，可以通过日志文件查看消息传递记录。默认日志路径通常为 `/var/log/mosquitto/mosquitto.log`。

```bash
sudo tail -f /var/log/mosquitto/mosquitto.log
```