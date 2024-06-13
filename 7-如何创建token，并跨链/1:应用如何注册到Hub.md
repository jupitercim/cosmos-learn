![img.png](img.png)



要将一个新的区块链（zone）注册到Cosmos Hub，实现与其他区块链的互操作性，主要通过IBC（Inter-Blockchain Communication Protocol）来进行。这涉及到多个步骤，包括链的配置、IBC客户端的创建、连接的建立以及通道的配置。以下是详细的步骤指南：

### 1. 准备工作

确保你的区块链（zone）已经准备好并符合IBC协议。你需要完成以下准备工作：

- **节点设置**：确保你的区块链节点已经设置并正常运行。
- **IBC模块集成**：确保你的区块链已经集成了IBC模块，通常包括轻客户端、连接和通道模块。
- **访问全节点**：确保你有访问源链和目标链全节点的权限，以便执行中继操作。

### 2. 创建IBC客户端

在你的区块链上创建一个指向Cosmos Hub的IBC客户端。IBC客户端用于跟踪对方链的状态。

```sh
gaiad tx ibc client create [client-type] [client-parameters] --from [your-key] --chain-id [your-chain-id]
```

示例（假设使用Tendermint客户端）：

```sh
gaiad tx ibc client create 07-tendermint \
  --client-state '{"@type":"/ibc.lightclients.tendermint.v1.ClientState","chain_id":"cosmoshub-4","trust_level":{"numerator":"1","denominator":"3"},"trusting_period":"336h","unbonding_period":"504h","max_clock_drift":"5m","frozen_height":{"revision_number":"0","revision_height":"0"},"latest_height":{"revision_number":"2","revision_height":"1024"},"proof_specs":[],"upgrade_path":[],"allow_update_after_expiry":false,"allow_update_after_misbehaviour":false}' \
  --from [your-key] \
  --chain-id [your-chain-id]
```

### 3. 创建IBC连接

在你的区块链和Cosmos Hub之间创建一个IBC连接。

```sh
gaiad tx ibc connection open-init \
  --client-id [your-client-id] \
  --counterparty-client-id [counterparty-client-id] \
  --from [your-key] \
  --chain-id [your-chain-id]
```

接着在Cosmos Hub上完成连接：

```sh
gaiad tx ibc connection open-try \
  --client-id [hub-client-id] \
  --counterparty-connection-id [your-connection-id] \
  --from [hub-key] \
  --chain-id cosmoshub-4
```

继续完成连接确认和连接打开：

```sh
gaiad tx ibc connection open-ack \
  --connection-id [your-connection-id] \
  --counterparty-connection-id [hub-connection-id] \
  --from [your-key] \
  --chain-id [your-chain-id]

gaiad tx ibc connection open-confirm \
  --connection-id [hub-connection-id] \
  --from [hub-key] \
  --chain-id cosmoshub-4
```

### 4. 创建IBC通道

在你的区块链和Cosmos Hub之间创建一个IBC通道。

```sh
gaiad tx ibc channel open-init \
  --port-id [your-port-id] \
  --channel-id [your-channel-id] \
  --connection-id [your-connection-id] \
  --from [your-key] \
  --chain-id [your-chain-id]
```

接着在Cosmos Hub上完成通道：

```sh
gaiad tx ibc channel open-try \
  --port-id [hub-port-id] \
  --channel-id [hub-channel-id] \
  --connection-id [hub-connection-id] \
  --from [hub-key] \
  --chain-id cosmoshub-4
```

继续完成通道确认和通道打开：

```sh
gaiad tx ibc channel open-ack \
  --port-id [your-port-id] \
  --channel-id [your-channel-id] \
  --counterparty-port-id [hub-port-id] \
  --counterparty-channel-id [hub-channel-id] \
  --from [your-key] \
  --chain-id [your-chain-id]

gaiad tx ibc channel open-confirm \
  --port-id [hub-port-id] \
  --channel-id [hub-channel-id] \
  --from [hub-key] \
  --chain-id cosmoshub-4
```

### 5. 运行Relayer

配置并运行一个Relayer来中继你的区块链和Cosmos Hub之间的IBC数据包。你可以使用多种Relayer软件，如[Go Relayer](https://github.com/cosmos/relayer)或[Hermes Relayer](https://github.com/informalsystems/ibc-rs)。

#### 示例配置文件（Go Relayer）

```yaml
chains:
  cosmoshub:
    chain-id: cosmoshub-4
    rpc-addr: https://rpc.cosmos.network:26657
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.01uatom
    trusting-period: 336h

  yourchain:
    chain-id: [your-chain-id]
    rpc-addr: [your-chain-rpc-addr]
    account-prefix: [your-account-prefix]
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.01ustake
    trusting-period: 336h

paths:
  yourchain-hub:
    src:
      chain-id: yourchain
      port-id: transfer
      channel-id: channel-0
    dst:
      chain-id: cosmoshub
      port-id: transfer
      channel-id: channel-0
    strategy:
      type: naive
```

运行Relayer：

```sh
rly start yourchain-hub
```

### 总结

通过以上步骤，你可以将你的区块链注册到Cosmos Hub，实现跨链互操作性。关键步骤包括创建IBC客户端、建立连接和通道，并运行Relayer来中继数据包。这些操作需要对区块链和IBC协议有一定的了解，并确保配置文件的正确性和节点的正常运行。
