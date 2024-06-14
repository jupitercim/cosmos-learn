---
title: "什么是IBC？"
order: 2
description: IBC协议介绍
tags:
  - 概念
  - ibc
---

# 什么是IBC？

<HighlightBox type="learning">

跨链通信协议（IBC）解决了区块链之间的通信问题，这在跨链生态系统中尤为重要。
<br/><br/>
在本节中，您将学习：

* 什么是IBC。
* IBC如何工作。
* IBC的安全保障。

</HighlightBox>

**[跨链通信协议（IBC）](https://ibcprotocol.dev/)** 是一种_处理两个区块链之间数据认证和传输的协议_。IBC **只需一组最小功能**，这些功能在[跨链标准（ICS）](https://github.com/cosmos/ibc/tree/master/spec/ics-001-ics-standard)中进行了规定。请注意，这些规范不限制网络拓扑或共识算法，因此IBC可以用于广泛的区块链或状态机。IBC协议提供了一种无需信任的跨链数据包传递方式，不像大多数需要信任的桥接技术。IBC的安全性归结于参与链的安全性。

IBC解决了一个广泛存在的问题：跨链通信。当交易所希望进行跨链交换时，这个问题存在于公共区块链中。在应用特定的区块链中，这个问题很早就出现了，每个资产可能都会从其专门构建的链上出现。在私有区块链的世界中，当需要与公共链或其他私有链通信时，跨链通信也是一个挑战。已经有一些私有区块链的IBC实现，例如[Hyperledger Fabric和Corda](https://www.hyperledger.org/blog/2021/06/09/meet-yui-one-the-new-hyperledger-labs-projects-taking-on-cross-chain-and-off-chain-operations)。

跨链通信在跨链生态系统中的应用特定区块链之间创造了高水平的横向可扩展性和交易终结性。这些设计特性为其他平台中的一些已知问题提供了有说服力的解决方案，如交易成本、网络容量和交易确认终结性。

## 区块链互联网

IBC对跨链生态系统中的应用特定区块链至关重要。它为需要彼此通信的两个不同链上的应用提供了标准通信通道。

<!-- @Cit: include relative link to Interchain Security content -->

大多数跨链应用在其专门构建的区块链上执行，运行自己的验证者集（至少在[跨链安全](https://informal.systems/2022/05/09/building-with-interchain-security)引入之前）。这些是使用Cosmos SDK构建的应用特定区块链。一个链上的应用可能需要与另一个区块链上的应用通信，例如，一个应用可以接受来自另一个区块链的代币作为支付方式。此级别的互操作性需要一种交换关于另一个区块链状态或交易数据的方法。

虽然可以构建桥接区块链并且确实存在，但它们通常是临时构建的。IBC为区块链提供了一个通用协议和框架，用于实现标准化的跨链通信。对于使用Cosmos SDK构建的区块链，这种功能是开箱即用的，但IBC协议并不限于使用跨链栈构建的链。

<HighlightBox type="info">

关于规范的更多详细信息将在下一节中介绍，但请注意IBC不仅限于Cosmos区块链。即使在某些要求最初未满足的情况下，也可以找到解决方案。例如，在“合并”之前，IBC已经提供了跨链生态系统和以太坊区块链之间的连接，“合并”见证了以太坊从工作量证明（PoW）模式向权益证明（PoS）模式的迁移。
<br/><br/>
由于PoW共识算法不确保终结性，而使用IBC的一个主要要求是确保终结性。因此，通过创建一个中继区，在达到一定的区块确认阈值后，将概率终结性视为确定性（不可逆）。此解决方案可以用于任何与PoW区块链的IBC连接。

</HighlightBox>

虽然应用特定区块链相对于通用区块链平台提供了更好的（横向）可扩展性，但通用链和通用虚拟机（如以太坊）的智能合约开发也有其自身的优势。IBC提供了一种将通用区块链和应用特定区块链的优势结合到统一总体设计中的方法。例如，它允许一个针对性能和可扩展性优化的Cosmos链使用起源于以太坊的资金，并可能在Corda分布式账本中记录事件；或者相反，一个Corda账本启动在跨链生态系统或以太坊中定义的基础资产的转移。

![区块链互联网](/academy/3-ibc/images/internetofchains.png)

通过IBC进行的跨链通信，可以实现独立和互操作链的去中心化网络，这些链之间可以交换信息和资产。这个“区块链互联网”带来了无缝扩展性的承诺。在跨链生态系统中，正在实施的愿景是拥有一个独立链的宇宙，它们都使用中继区作为跨链生态系统和其外部链之间的桥梁，并通过枢纽连接所有链。这些组成了区块链互联网。

## IBC的高级概述

传输层（TAO）提供了建立安全连接和验证数据包所需的基础设施。应用层构建在传输层之上，定义了数据包应如何打包和解释，以便发送和接收链。

<HighlightBox type="remember">

IBC的伟大承诺是提供一个可靠的、无需许可的和通用的基础层（允许安全传递数据包），同时通过将应用设计（解释和处理数据包数据）移到更高层来实现可组合性和模块化，从而实现关注点分离。

</HighlightBox>

这种分离反映在ICS的类别定义中，可以在[一般ICS定义](https://github.com/cosmos/ibc/blob/master/spec/ics-001-ics-standard/README.md)中看到：

* **IBC/TAO:** 定义数据包的传输、认证和排序的标准，即基础设施层。在ICS中，这包括_Core_、_Client_和_Relayer_类别。
* **IBC/APP:** 定义传输层上数据包应用处理程序的标准。这些包括但不限于可替代代币转移（ICS-20）、NFT转移（ICS-721）和跨链账户（ICS-27），可以在ICS的_App_类别中找到。

以下图示展示了IBC的工作原理：

![IBC概述 - 两个连接的链](/academy/3-ibc/images/ibcoverview.png)

请注意图中三个关键元素：

* 链依赖于中继器进行通信。中继器算法（[ICS-18](https://github.com/cosmos/ibc/tree/master/spec/relayer/ics-018-relayer-algorithms)）是IBC的“物理”连接层：负责在运行IBC协议的两个链之间中继数据的链外进程，通过扫描每个链的状态，构建适当的数据报，并按照协议允许的方式在对方链上执行它们。
* 许多中继器可以服务于一个或多个通道来在链之间发送消息。
* 每个中继器的一侧使用另一个链的轻客户端快速验证传入消息。

<HighlightBox type="info">

如果您对IBC协议的另一个概述感兴趣，以下视频中Tendermint Core的工程经理Callum Waters介绍了允许无数主权区块链之间互操作的方法，以及如何构建IBC兼容应用。

<YoutubePlayer videoId="OSMH5uwTssk"/>

</HighlightBox>

### IBC/TAO - 传输层

在图中，TAO类别中的ICS定义的关系得到了说明——箭头说明了需求。

简单来说，传输层包括：

* **轻客户端** - [ICS-2](https://github.com/cosmos/ibc/tree/master/spec/core/ics-002-client-semantics), [ICS-6](https://github.com/cosmos/ibc/tree/master/spec/client/ics-006-solo-machine-client), [ICS-7](https://github.com/cosmos/ibc/tree/master/spec/client/ics-007-tendermint-client), [ICS-8](https://github.com/cosmos/ibc/tree/master/spec/client/ics-008-wasm-client), [ICS-9](https://github.com/cosmos/ibc/tree/main/spec/client/ics-009-loopback-cilent): IBC客户端是由唯一的客户端ID标识的轻客户端。IBC客户端跟踪其他区块链的共识状态和所需的证明规范，以便正确地根据客户端的共识状态验证证明。一个客户端可以与任意数量的连接相关联。
* **连接** - [ICS-3](https://github.com/cosmos/ibc/tree/master/spec/core/ics-003-connection-semantics): 一旦建立，连接负责促进所有跨链的IBC状态验证。一个连接可以与任意数量的通

道相关联。
* **通道** - [ICS-4](https://github.com/cosmos/ibc/tree/master/spec/core/ics-004-channel-semantics): 通道使用唯一的标识符（`<port-id>:<channel-id>`），用于承载跨链数据包传输。每个通道都是为单个应用使用的。

### IBC/APP - 应用层

传输层是IBC的“工作引擎”。更高层允许应用设计者定义如何解释IBC数据包（即如何在链B上应用从链A接收的数据包）。应用设计的标准由在[ICS App](https://github.com/cosmos/ibc/tree/master/spec/app)中定义的ICS标准记录。尽管应用标准数量不断增加，但常见的示例如下：

* **ICS-20** - [代币转移](https://github.com/cosmos/ibc/tree/master/spec/app/ics-020-fungible-token-transfer): ICS-20应用定义了如何通过IBC发送数据包以在链之间转移同质化代币。
* **ICS-27** - [跨链账户](https://github.com/cosmos/ibc/tree/master/spec/app/ics-027-cross-chain-account): ICS-27应用定义了如何通过IBC发送数据包以在一个链上注册和控制另一个链上的账户。
* **ICS-28** - [跨链账户](https://github.com/cosmos/ibc/tree/master/spec/app/ics-028-cross-chain-account): ICS-28应用定义了如何通过IBC发送数据包以在链之间发送和处理非同质化代币（NFT）。

<HighlightBox type="learning">

在下一节中，您将深入了解IBC/TAO传输层的工作原理和细节。

</HighlightBox>

---

