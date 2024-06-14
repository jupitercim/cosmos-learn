## 概要

本标准文档规定了跨两个不同链上的模块通过IBC通道转移同质化代币的包数据结构、状态机处理逻辑和编码细节。所提出的状态机逻辑允许在无需许可的通道开通情况下，安全地处理多链代币。此逻辑构成了一个“同质化代币转移桥接模块”，在IBC路由模块和主状态机上的现有资产追踪模块之间进行接口连接。

## 动机

连接在IBC协议上的一组链的用户可能希望在另一条链上使用一条链上发行的资产，或许是为了利用额外的功能，如交易或隐私保护，同时保持与原发行链上资产的同质化。此应用层标准描述了一种协议，用于在连接IBC的链之间转移同质化代币，保留资产的同质化，保留资产所有权，限制拜占庭故障的影响，并且不需要额外的权限。

## 定义

IBC处理器接口和IBC路由模块接口分别在ICS 25和ICS 26中定义。

## 期望属性

- 保持同质化（双向锚定）。
- 保持总供应量（在单一源链和模块上恒定或通货膨胀）。
- 无需许可的代币转移，无需白名单连接、模块或面额。
- 对称性（所有链都实现相同的逻辑，协议内不区分集线器和区域）。
- 故障隔离：防止由于链B的拜占庭行为导致链A上代币的拜占庭膨胀（虽然任何将代币发送到链B的用户可能会面临风险）。

## 技术规范

### 数据结构

只需要一种包数据类型：`FungibleTokenPacketData`，它指定了面额、金额、发送账户和接收账户；或者`FungibleTokenPacketDataV2`，它指定了在发送者和接收者之间传输的多个代币。支持v2的链可以可选地将v1包转换为仍在版本1上的通道。

```javascript
interface FungibleTokenPacketData {
  denom: string
  amount: uint256
  sender: string
  receiver: string
  memo: string
}

interface FungibleTokenPacketDataV2 {
  tokens: []Token
  sender: string
  receiver: string
  memo: string
}

interface Token {
  denom: string // base denomination
  trace: []string
  amount: uint64
}
```

当代币通过ICS 20协议在链之间传输时，它们开始累积转移过的通道记录。这些信息被编码到代币的`trace`字段中。

ICS 20代币追踪通过形如`{ics20Port}/{ics20Channel}`的列表表示，其中`ics20Port`和`ics20Channel`是当前链上资金存在的ICS 20端口和通道。端口和通道对表示资金以前通过的通道。实现负责正确解析IBC追踪信息并将其编码到最终的链上面额中，以确保通过不同路径发送的相同基本面额不会被视为同质化。

发送链可以作为源区或汇区。当一条链通过一个端口和通道发送代币，并且这个端口和通道不等于最后前缀的端口和通道对时，它充当源区。当代币从源区发送时，目标端口和通道将在接收代币后添加到追踪中，为代币记录增加一个跳跃。当一条链通过一个端口和通道发送代币，并且这个端口和通道等于最后前缀的端口和通道对时，它充当汇区。当代币从汇区发送时，追踪中的第一个元素，即最后添加到追踪中的端口和通道对将被移除，撤销代币记录中的最后一个跳跃。更完整的解释在ibc-go实现中。

以下序列图示例了多链代币转移的动态。这一过程封装了在链A、链B和链C之间转移代币的复杂步骤，转移周期以相同链为起点和终点，顺序为A -> B -> C -> A -> C -> B -> A。

### 转移示例

确认数据类型描述了转移是成功还是失败，以及失败的原因（如果有的话）。

```javascript
type FungibleTokenPacketAcknowledgement = FungibleTokenPacketSuccess | FungibleTokenPacketError;

interface FungibleTokenPacketSuccess {
  // This is binary 0x01 base64 encoded
  result: "AQ=="
}

interface FungibleTokenPacketError {
  error: string
}
```

请注意，`FungibleTokenPacketData`和`FungibleTokenPacketAcknowledgement`在序列化为包数据时必须进行JSON编码（而不是Protobuf编码）。还要注意，当转换为JSON时，`uint256`是字符串编码的，但必须是格式为`[0-9]+`的有效十进制数。

同质化代币转移桥接模块在状态中跟踪与特定通道关联的托管地址。`ModuleState`的字段被假定为范围内的。

```javascript
interface ModuleState {
  channelEscrowAddresses: Map<Identifier, string>
}
```

### 子协议

本文描述的子协议应在具有银行模块和IBC路由模块访问权限的“同质化代币转移桥接”模块中实现。

#### 端口和通道设置

创建模块时（可能是在区块链本身初始化时）必须调用设置函数，绑定到适当的端口并创建一个由模块拥有的托管地址。

```javascript
function setup() {
  capability = routingModule.bindPort("transfer", ModuleCallbacks{
    onChanOpenInit,
    onChanOpenTry,
    onChanOpenAck,
    onChanOpenConfirm,
    onChanCloseInit,
    onChanCloseConfirm,
    onRecvPacket,
    onTimeoutPacket,
    onAcknowledgePacket,
    onTimeoutPacketClose
  })
  claimCapability("port", capability)
}
```

一旦调用了设置函数，就可以通过IBC路由模块在不同链上的同质化代币转移模块实例之间创建通道。

管理员（具有在主状态机上创建连接和通道的权限）负责设置与其他状态机的连接，并创建与其他链上此模块（或支持此接口的其他模块）实例的通道。本规范仅定义包处理语义，并以这种方式定义，使得模块本身不需要担心在任何时间点上可能存在或不存在的连接或通道。

### 路由模块回调

#### 通道生命周期管理

机器A和B仅在以下情况下接受来自其他机器上任何模块的新通道：

- 被创建的通道是无序的。
- 版本字符串是`ics20-1`或`ics20-2`。

```javascript
function onChanOpenInit(
  order: ChannelOrder,
  connectionHops: [Identifier],
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  counterpartyPortIdentifier: Identifier,
  counterpartyChannelIdentifier: Identifier,
  version: string) => (version: string, err: Error) {
  // 仅允许无序通道
  abortTransactionUnless(order === UNORDERED)
  // 断言版本是"ics20-1"或"ics20-2"或为空
  // 如果为空，我们将默认传输版本返回给核心IBC
  // 作为此通道的版本
  abortTransactionUnless(version === "ics20-2" || version === "ics20-1" || version === "")
  // 分配托管地址
  channelEscrowAddresses[channelIdentifier] = newAddress(portIdentifier, channelIdentifier)
  if version == "" {
    // 默认使用最新支持的版本
    return "ics20-2", nil
  }
  // 如果版本不为空且在支持的范围内，我们将返回该版本
  return version, nil 
}
```

```javascript
function onChanOpenTry(
  order: ChannelOrder,
  connectionHops: [Identifier],
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  counterpartyPortIdentifier: Identifier,
  counterpartyChannelIdentifier: Identifier,
  counterpartyVersion: string) => (version: string, err: Error) {
  // 仅允许无序通道
  abortTransactionUnless(order === UNORDERED)
  // 断言对方版本是"ics20-1"或"ics20-2"
  abortTransactionUnless(counterpartyVersion === "ics20-1" || counterpartyVersion === "ics20-2")
  // 分配托管地址
  channelEscrowAddresses[channelIdentifier] = newAddress(portIdentifier, channelIdentifier)
  // 返回与对方版本相同的版本，只要我们支持它
  return counterpartyVersion, nil
}
```

```javascript
function onChanOpenAck(
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  counterpartyChannelIdentifier: Identifier,
  counterpartyVersion: string) {
  // 端口已被验证
  // 断言对方选择的版本与我们的版本相同
  channel = provable