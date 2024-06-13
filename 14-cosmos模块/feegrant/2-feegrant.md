---
title: "了解 Feegrant 模块"
order: 3
description: 使用 Cosmos SDK 的 feegrant 模块，允许一个账户（授权人）为另一个账户（被授权人）支付费用。
tags:
  - 教程
  - cosmos-sdk
  - 运维
---

# 了解 Feegrant 模块

[`feegrant`](https://docs.cosmos.network/v0.45/modules/feegrant/) 模块允许授权人（用户、合约或模块）在被授权人希望在区块链上广播交易时为其支付费用。授权人保留对其代币的完全访问权，并且可以随时撤销授权。

## 使用 feegrant 授予额度

`feegrant` 模块的一个常见用例是_改进的用户引导体验_，因为新用户无需在开始与区块链或智能合约交互之前获取代币。

`feegrant` 模块实现了两种[费用额度类型](https://docs.cosmos.network/v0.45/modules/feegrant/01_concepts.html#fee-allowance-types)：

* `BasicAllowance`
  被授权人使用授权人的账户支付费用。额度可以设置一次性限制、到期时间或没有限制。
* `PeriodicAllowance`
  被授权人使用授权人的账户支付费用。额度有一个定期重置的限制。

在本教程中，您将为区块链设置两个代币：一个用于支付费用的默认代币 `stake`，另一个用于发送给朋友的代币 `kudos`。

* 您将学习如何使用 Cosmos SDK 的模拟应用程序 (`simapp`) 启动单节点网络。
* 设置 Alice 作为验证者。
* Bob 将是被授权人，接收一个 `BasicAllowance`，允许 Bob 发送 `kudos` 代币给 Alice，即使 Bob 没有 `stake` 来支付费用。
* Alice 将是授权人，授予 Bob 一个 `BasicAllowance`。

## 要求

在开始本教程之前，您需要安装 `simd` 二进制文件。

克隆 `cosmos-sdk` 仓库：

```sh
$ git clone https://github.com/cosmos/cosmos-sdk
```

切换目录并检出 `v0.44.0`：

```sh
$ cd cosmos-sdk && git checkout v0.44.0
```

安装 `simd` 二进制文件：

```sh
$ make install
```

检查安装是否成功：

```sh
$ simd version
```

控制台输出版本号 `0.44.0`。

## 配置

<HighlightBox type="tip">

如果您之前使用过 `simd`，您可能已经在主目录中有一个 `.simapp` 目录。为了保留以前的数据，可以将目录保存到另一个位置，或者在接下来的指令中使用 `--home` 标志并指定不同的目录。如果您不想保留以前的数据，请删除以前的目录 (`rm -rf ~/.simapp`)。

</HighlightBox>

运行以下命令配置 `simd` 二进制文件。

设置链 ID：

```sh
$ simd config chain-id demo
```

设置[密钥环后端](https://docs.cosmos.network/main/run-node/keyring.html#setting-up-the-keyring)：

```sh
$ simd config keyring-backend test
```

## 密钥设置

您需要为您的用户创建一些测试密钥。

为 Alice 添加一个密钥，Alice 是验证者：

```sh
$ simd keys add alice
```

为 Bob 添加一个密钥，Bob 是被授权人：

```sh
$ simd keys add bob
```

如果您想查看密钥的概览，使用：

```sh
$ simd keys list
```

<HighlightBox type="tip">

为了避免复制和粘贴用户地址，现在是将用户密钥导出到变量中以便在本教程中使用的好时机。

</HighlightBox>

```sh
$ export ALICE=$(simd keys show

alice --address)
$ export BOB=$(simd keys show bob --address)
```

## 链设置

以下命令使用模拟应用程序 (`simapp`) 设置链。

初始化节点：

```sh
$ simd init test --chain-id demo
```

Alice 是您的验证者。将 Alice 和初始余额添加到创世文件中：

```sh
$ simd add-genesis-account alice 5000000000stake --keyring-backend test
```

将 Bob 和初始余额添加到创世文件中：

```sh
$ simd add-genesis-account bob 2000kudos --keyring-backend test
```

<HighlightBox type="note">

注意，Bob 只有 `kudos` 代币，无法支付可能需要的任何费用。

</HighlightBox>

生成一个交易以将 Alice 添加到初始验证者集中：

```sh
$ simd gentx alice 1000000stake --chain-id demo
```

将验证者交易添加到创世文件中：

```sh
$ simd collect-gentxs
```

## 启动链

现在，您可以在本地机器上启动单节点网络。

启动链：

```sh
$ simd start
```

## 授予额度

在 Bob 能够将 `kudos` 发送给 Alice 之前，您必须为 Bob 设置一个额度，以便 Alice 支付任何交易可能产生的燃料费用。

`BasicAllowance` 是一个允许被授权人在达到 `spend_limit` 或 `expiration` 之前使用费用的许可。打开一个新的终端窗口，并创建一个花费限制为 `100000stake` 且没有到期日期的额度：

```sh
$ simd tx feegrant grant $ALICE $BOB --from alice --spend-limit 100000stake
```

查看额度：

```sh
$ simd query feegrant grants $BOB
```

## 发送代币

首先，检查 Alice 和 Bob 的余额。验证初始余额提供了一个基线，以便稍后确认您的交易是否成功：

```sh
$ simd query bank balances $ALICE
$ simd query bank balances $BOB
```

<HighlightBox type="note">

注意，Alice 现在有 `4999000000stake`，因为她在链设置过程中绑定了 `1000000stake` 成为验证者。

</HighlightBox>

任何使用 `tx` 命令发送的交易都可以使用 `--fee-account` 标志来指定一个账户来支付费用。

将 `kudos` 代币从 Bob 发送给 Alice，同时由 Alice 支付费用：

```sh
$ simd tx bank send $BOB $ALICE 100kudos --from bob --fee-account $ALICE --fees 500stake
```

再次查看余额：

```sh
$ simd query bank balances $ALICE
$ simd query bank balances $BOB
```

注意，Alice 比之前少了 `500stake`。这 `500stake` 已经添加到 Bob 签署的交易中。

再次查看额度：

```sh
$ simd query feegrant grants $BOB
```

<HighlightBox type="note">

注意，`spend_limit` 已经减少，现在 Bob 剩余 `99500stake` 可以用来支付费用。

</HighlightBox>

## 撤销额度

授权人可以使用 `revoke` 命令撤销被授权人的额度。

撤销额度：

```sh
$ simd tx feegrant revoke $ALICE $BOB --from alice
```

查看额度：

```sh
$ simd query feegrant grants $BOB
```

## 🎉 恭喜 🎉

通过完成本教程，您已经学会了如何使用 `feegrant` 模块！

<HighlightBox type="info">

想看 x/feegrant 和 x/authz 的演示吗？在以下视频中，Parity Technologies 的核心开发者 Amaury Martiny 和 Vitwit 的软件开发者 Likhita Polavarapu 演示了这两个模块的显著用户体验优势，以及如何将它们集成到区块链应用程序中。

<YoutubePlayer videoId="g6t5ZJSQDus"/>

</HighlightBox>

<HighlightBox type="synopsis">

总结，本节探讨了：

* 如何配置和使用模拟应用程序 (simapp)。
* 如何创建一个额度。
* 如何在授权人支付费用的情况下进行交易。
* 如何撤销一个额度。

</HighlightBox>

`feegrant` 模块还有很多其他功能可以探索。您可以添加允许的消息列表、设置到期日期，以及设置在花费限制后重置的时间间隔。要了解更多关于 `feegrant` 模块和不同类型额度的信息，请参阅 Cosmos SDK [Feegrant Module](https://docs.cosmos.network/v0.45/modules/feegrant/) 文档。