# Rollup集成指南

## 背景

以下是针对寻求与IBC集成的Rollup框架的指南。Rollup是依赖第三方区块链进行数据可用性（DA）的去中心化应用程序，并可选择用于结算的一种形式。Rollup的共识机制与主权区块链有重要区别。Rollup的区块和排序共识由它们被发布到第三方账本（即DA层）的顺序定义。由于第三方账本本身不执行交易和构建Rollup应用程序状态，因此Rollup可能具有结算机制。Rollup架构有两种类型：乐观型和零知识（ZK）。ZK Rollup提交证明表明报告的应用哈希正确地由块中包含的交易构造而来，因此一旦在DA层上确定完成，Rollup块和头可以被信任为合法的。而乐观型Rollup则依赖于第三方观察者，可以向结算层提交证明，证明Rollup未正确地从发布的交易计算出应用哈希。这要求结算层能够执行Rollup状态机。DA层和结算层可能是不同的区块链，也可能是同一区块链。

本指南不旨在成为正式规范或Interchain标准。由于Rollup及其底层数据可用性和结算层的架构差异巨大：从ZK Rollup到具有独立数据可用性和结算层的乐观型Rollup，再到主权Rollup，因此不可能编写一个全面规定的客户端以涵盖所有这些情况。因此，本指南旨在突出IBC客户端功能，这些功能受Rollup特定特性影响，并解释在每个功能中必须执行的操作，以考虑Rollup独特属性的位置。Rollup轻客户端开发人员应使用本文档作为设计其轻客户端的起点，以确保在适当位置考虑Rollup特定逻辑。

## 定义

**执行层或Rollup**：这是Rollup区块链本身。它执行Rollup应用程序，并从底层层次（例如DA层）派生其共识和安全性。Rollup客户端是跟踪Rollup区块链的轻客户端。

**序列化器**：这是收集用户交易并创建新Rollup块的参与者。序列化器必须将这些块发布到数据可用性层。由于Rollup的安全性由数据可用性和结算层支持，因此序列化器不需要像主权验证者集那样去中心化，甚至可以是单个操作员。某些Rollup架构甚至可能是“无序列化器”的，此时任何参与者都可以将新块发布到数据可用性层。

**数据可用性层（DA层）**：这是Rollup块生成者必须发布其块的账本。因此，任何Rollup用户都可以从DA层下载Rollup区块链。因此，数据可用性层提供了Rollup块及其中包含的交易的可用性保证。由于数据可用性层是区块链，并具有明确定义的排序，因此可以从其在数据可用性层上的排序派生Rollup块的排序（即包含的交易的协商一致性）。DA客户端是跟踪数据可用性区块链的轻客户端。

**结算层**：结算层是解决发布的Rollup状态正确性争议的地方。除了包含的交易外，Rollup块生成者还必须发布应用新包含的交易到先前Rollup状态的状态哈希。如果Rollup块生成者发布了错误的应用哈希以及任何观察者都可以向结算层提交欺诈证明来争议错误的应用哈希。此时，结算层必须验证欺诈证明；通常通过一个欺诈证明游戏，该游戏要求块生成者和欺诈提交者在结算层能够执行相关逻辑之前，缩小到争议的执行结果。如果欺诈成立，结算层必须标记欺诈块为无效。此块及其后续基于它构建的块将从Rollup的区块链历史中无效并删除。结算层是可选的，因为某些Rollup架构不涉及结算。例如，Celestia Rollup是“主权Rollup”，因此全节点和Rollup点对点网络本身负责执行块并传播欺诈证明。此外，结算层可能是与DA层相同的账本，也可能是完全不同的账本。结算客户端是跟踪结算层区块链的轻客户端。

**主权Rollup**：主权Rollup将其块发布到数据可用性层，但不依赖于任何其他区块链以确保正确性（即结算）。因此，Rollup节点从数据可用性层派生共识和排序，但必须自行执行交易以验证正确性或从Rollup点对点网络获得欺诈证明。

**乐观型Rollup**：乐观型Rollup将其块发布到数据可用性层，并依赖于能够由Rollup观察者提交的结算层来裁决欺诈证明。因此，Rollup块“乐观地”被接受，而无需立即保证正确性，但只有在未提交任何成功挑战来证明头部的正确性后，它们才被认为是安全和最终的。

**ZK Rollup**：ZK Rollup具有表示其状态机的零知识电路。因此，Rollup块生成者可以提交一个ZK-SNARK证明，证明提交的应用哈希确实是从块中包含的交易中应用的正确结果。因此，无需结算层或欺诈窗口。一旦验证了ZK证明，块就可以被信任并最终化。

## `verifyClientMessage`

为了验证Rollup的新头部，Rollup客户端必须还能够验证头部（和相关块）在DA层的包含。因此，Rollup客户端的更新逻辑**必须**能够调用相关DA客户端的验证。在验证Rollup自身的共识机制之后（如果存在的话），它验证在数据可用性层中头部和块数据。然而，简单地证明包含并不够，我们必须确保我们证明的数据是有效的；即数据不仅仅是被包含在内，而是以Rollup架构预期的方式被包含。在下面的示例中，我们检查块数据是否散列到头部的`txHash`。

ZK Rollup可以在提交时验证头部的正确性，因为Rollup客户端可以嵌入一个证明电路，该电路可以验证中继人提交的头部是正确的。另一方面，乐观型Rollup不能立即信任头部在提交时，因为稍后可能会证明头部是欺诈的。因此，头部可以存储，但必须等待欺诈期过去，没有成功的挑战来证明头部的

正确性，然后才能最终化并用于证明验证。

```typescript
function verifyClientMessage(clientMessage: ClientMessage) {
  switch typeof(clientMessage) {
    case Header:
      verifyHeader(clientMessage)
    case Misbehaviour:
      // 这完全是Rollup特定的，因此在此处未指定
      // Rollup的不良行为验证规范
      // 完全在checkForMisbehaviour中描述
  }
}

function verifyHeader(clientMessage: ClientMessage) {
  clientState = provableStore.get("clients/{clientMessage.clientId}/clientState")
  header = Header(clientMessage)

  // 注意：未包含反序列化逻辑
  // 如果存在的话，验证头部与Rollup自身的共识机制（例如验证序列化器签名）
  verifySignatures(header, clientSequencers)

  // 我们必须断言块数据与头部关联正确
  // 这是特定于Rollup头部和块架构的
  // 以下仅是可能验证的示例
  assert(hash(header.blockData) === header.txHash)

  // 除了验证Rollup自身的共识机制之外，
  // 我们必须确保头部和关联的块数据存储在DA层中。
  // 期望路径、存储的头部和数据都是Rollup特定的，
  // 在本文档中未指定为未指定的函数。
  // 但是路径应引用此处指定的Rollup的唯一命名空间
  // 以及Rollup的链ID和唯一高度
  daClient = getClient(clientState.DALayer)
  verifyMembership(
    daClient,
    header.DAProofHeight,
    0,
    0,
    header.DAHeaderProof,
    DAHeaderPath(clientState.chainId, header.height),
    header)
  verifyMembership(
    daClient,
    header.DAProofHeight,
    0,
    0,
    header.DABlockDataProof,
    DABlockDataPath(clientState.chainID, header.height),
    header.blockData)

  // 如果Rollup是ZK Rollup，则我们可以立即验证其正确性。
  // 否则，提交的Rollup头部的正确性取决于在欺诈期内没有提交有效证明
  // （请参阅不良行为逻辑）
  prove(client.ZKProvingCircuit, header.zkProof)
}
```

## `updateState`

Rollup的`updateState`函数与典型客户端的工作方式相同，尽管乐观型Rollup客户端关键是存储创建共识状态时的提交时间，以便我们可以验证欺诈期是否已过。

```typescript
function updateState(clientMessage: ClientMessage) {
  // 略去编组逻辑
  header = Header(clientMessage)
  consensusState = ConsensusState{header.timestamp, header.appHash}

  provableStore.set("clients/{clientMessage.clientId}/consensusStates/{header.GetHeight()}", consensusState)

  // 为了欺诈证明等待期，创建共识状态和当前时间之间的映射
  provableStore.set("clients/{clientMessage.clientId}/processedTimes/{header.GetHeight()}", currentTimestamp())
}
```

## `checkForMisbehaviour`

Rollup架构中的不良行为验证与传统共识机制中的目的不同。

传统的共识机制（如权益证明）依赖于排序的自我依赖。因此，我们必须有机制来检测共识集是否违反排序规则。例如，在Tendermint中，不良行为验证检查头部时间是否单调增加，以及每个高度是否仅存在一个有效头部。

然而，对于Rollup，排序是从数据可用性层派生的。因此，即使在Rollup共识中存在违反，也可以通过DA层和Rollup的共识规则解决。例如，即使序列化器在相同高度签署多个块，标准块是提交到DA层的第一个块。

因此，只要验证方法正确编码了Rollup架构的共识规则（例如，确保提交的头部是给定高度的最早头部），则无需验证Rollup共识的不良行为。共识从DA层派生，因此如果由于不良行为而冻结DA客户端，这也应该阻止Rollup客户端中的证明验证。

相反，对于Rollup而言，最相关的不良行为是在应用层面，因为交易由序列化器执行，但不由底层数据可用性层执行。对于ZK Rollup，应用已经被证明是正确的，因此不需要应用不良行为验证。然而，乐观型Rollup必须提供能够使链下进程提交证明，证明在块中的交易计算中提交的应用哈希是错误的，即欺诈证明。

乐观型欺诈证明验证器或证明电路应该作为智能合约实现，因为欺诈证明者依赖于应用状态机本身，而不是共识机制。因此，每个Rollup实例都需要自己的欺诈证明者。直接将每个欺诈证明者编码到客户端中要求为每个Rollup实例进行不同的实现。相反，调用独立的智能合约允许客户端在所有实例中重复使用，并允许为新的Rollup应用上传新的欺诈证明者。

```typescript
// 乐观型Rollup欺诈证明
// 不良行为必须与Rollup的高度相关联
function checkForMisbehaviour(clientMessage: ClientMessage) {
  // 略去解码逻辑
  misbehaviour = Misbehaviour(clientMessage)
  clientId = clientMessage.clientId
  clientState = provableStore.get("clients/{clientMessage.clientId}/clientState")

  // 如果Rollup具有结算层，我们可以将欺诈证明游戏委托给结算层
  // 并简单地验证结算客户端已成功证明给定不良行为
  if clientState.settlementLayer == nil {
    // 欺诈证明器在此是一个合约，因此相同的Rollup客户端实现可能
    // 为每个不同的状态机初始化不同的欺诈证明器合约
    fraudProverContract = getFraudProver(clientId)
    fraudProverContract.verifyFraudProof(misbehaviour)
  } else {
    // 为了使用结算客户端，必须在特定路径上存储一个特定的标记值
    // 以供给定Rollup和高度的路径使用
    misbehavingHeight = getHeight(misbehaviour)
    settlementClient = getClient(clientState.settlementLayer)
    misbehaviourPath = getMisbehaviourPath(clientId, misbehavingHeight)
    settlementClient.verifyMembership(misbehaviour.proofHeight, 0, 0, misbehaviour.proof, misbehaviourPath, MISBEHAVIOUR_SUCCESS_VALUE)
  }
}
```

## `updateStateOnMisbehaviour`

不良行为更新也依赖于Rollup架构。在主权权益证明链上，如果违反了共识规则，通常没有回退机制，因为对链的信任完全破坏，除非在协议外的社会共识下重启具有新验证器集的链。因此，对于主权链，接收到有效不良行为后，客户端应该简单地被禁用。

另一方面，Rollup确实在数据可用性和结算层上有一个后备层。例如，结算层可以验证块无效并简单地删除，从而强制块可以继续使用有效状态，因为结算层可以继续从链历史中删除无效块。类似地，