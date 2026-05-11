# 快速开始

ThingsBoard 是一个开源应用，可以在docker中部署运行，提供物联网设备管理、数据收集、处理和可视化功能。

## 安装部署

### Docker 部署
官方文档中有docker部署指南，其中将thingsboard和postgresql数据库作为两个service进行部署，使用docker-compose管理。

也可以使用 thingsboard/tb-postgres 镜像包含了ThingsBoard和PostgreSQL数据库，使用该镜像可以简化部署过程，无需单独配置数据库。
```bash
services:
  thingsboard:
    image: thingsboard/tb-postgres    # 该镜像包含了ThingsBoard和PostgreSQL数据库
    container_name: thingsboard
    restart: always
    ports:
      - "8010:9090"   # ThingsBoard Web UI
      - "7070:7070"
      - "18833:1883"    # MQTT
      - "5683-5688:5683-5688/udp"
    volumes:
      - ./mytb-data:/data
      - ./mytb-logs:/var/log/thingsboard
    stdin_open: true # 对应 -i
    tty: true        # 对应 -t
```


## 使用说明
### 登录
访问http://localhost:8010，默认有三个用户：

* System Administrator: sysadmin@thingsboard.org / sysadmin
* Tenant Administrator: tenant@thingsboard.org / tenant
* Customer User: customer@thingsboard.org / customer

### 创建设备
登录后可以创建设备，设备是ThingsBoard中用于表示物联网设备的数字化实体。创建设备后，thingsboard会为设备分配一个唯一的访问令牌（access token），该令牌用于设备与ThingsBoard平台之间的通信和数据交换。

### 设备通信
实际的物理设备可以通过MQTT、HTTP或CoAP协议与ThingsBoard平台进行通信。物理设备使用访问令牌进行身份验证，并将数据发送到ThingsBoard平台，平台会处理这些数据并提供可视化和分析功能。

### 数据可视化
ThingsBoard提供了丰富的数据可视化功能，可以创建仪表板（dashboard）来展示设备数据。仪表板可以包含各种小部件（widget），如图表、表格、地图等，用于实时显示设备数据和状态。

### 规则链
规则链（Rule Chain）是ThingsBoard中的一个重要概念，用于定义设备数据处理和事件响应的逻辑。通过规则链，可以设置条件和操作来处理设备数据，例如触发警报、发送通知、存储数据等。

# API/组件 详解

## 规则链

规则链是用户编写业务逻辑的地方，在这里配置IOT设备之间的协同工作,以及IOT设备和外部系统之间的交互。

用户可以创建多个规则链，但是只能标记一个为Root Rule Chain，作为默认规则链。如果设备没有指定规则链，就会使用Root Rule Chain进行处理；如果设备配置了profile，并且profile中指定了规则链，那么就会使用profile中指定的规则链进行处理。

### Input节点
Input节点是规则链中的第一个节点，接受`系统事件`、`设备事件`、`RPC请求`等输入数据，并将其传递给下一个节点进行处理。通常业务逻辑只处理`设备事件`。

Thingsborad中要求device 必须使用access token进行认证，以及只能向固定的topic发送数据。
* 如果access token不正确，则会被拒绝访问，无法发送数据。
* 如果发送数据的topic不正确，但access token正确，则会修改device的状态，但实际数据不会被处理。

