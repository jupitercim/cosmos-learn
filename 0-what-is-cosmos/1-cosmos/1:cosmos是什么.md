# COSMOS
## 一个分布式账本网络

Cosmos 项目有着宏伟的使命 —— 创建一个解决加密货币和区块链领域长期存在问题的分布式账本网络。

Cosmos 是一个由独立并行区块链组成的网络，每个区块链都由传统的 BFT 共识算法（如 Tendermint）驱动。这个网络中的第一个区块链将是 Cosmos Hub。Cosmos Hub 通过一种新颖的区块链间通信协议与许多其他区块链（或“区域”）连接在一起。Cosmos Hub 跟踪多种代币类型，并记录每个连接区域中的代币总量。代币可以在区域之间安全快速地转移，无需在区域之间进行流动性交换，因为所有区域间的币种转移都通过 Cosmos Hub 进行。

这种架构解决了当前区块链领域面临的许多问题，包括应用程序互操作性、可扩展性和无缝升级。例如，来自 Bitcoind、Go-Ethereum、CryptoNote、ZCash 或任何区块链系统的区域可以插入 Cosmos Hub。这些区域使得 Cosmos 能够无限扩展以满足全球交易需求。而且，区域非常适合分布式交易所，这也将得到支持。

Cosmos 不仅仅是一个单一的分布式账本，Cosmos Hub 也不是一个封闭的园地或宇宙的中心。我们正在设计一个开放的分布式账本网络协议，它可以作为未来金融系统的新基础，基于密码学、良好的经济学、共识理论、透明度和问责制原则。

<img src="https://raw.githubusercontent.com/cosmos/cosmos/master/images/ex_zone.png" height="100"/>

Cosmos 区域是一个分布式账本或区块链。每个区域可以具有不同的交易逻辑和经济/治理/安全策略。

<img src="https://raw.githubusercontent.com/cosmos/cosmos/master/images/ex_zone_ibc.png" height="100"/>

我们开发了一种技巧，允许区域之间直接通信。它基于像 Tendermint 这样的经典 BFT 算法。

<img src="https://raw.githubusercontent.com/cosmos/cosmos/master/images/ex_evm_sharding.png" height="240"/>

通过这种技术，我们可以通过构建一个共同的中心区块链使以太坊扩展。所有区域间的代币移动都将通过中心枢纽进行。

<img src="https://raw.githubusercontent.com/cosmos/cosmos/master/images/ex_evm_upgrading.png" height="60"/>

通过此技术，升级 EVM 将变得无缝，风险较低，不太容易发生有争议的硬分叉。任何人都可以插入一个更好的、升级后的智能合约系统。

<img src="https://raw.githubusercontent.com/cosmos/cosmos/master/images/ex_dist_exchange.png" height="240"/>

我们可以引入其他区块链，并在其自己的区域上建立一个分布式交易所。这可能比中心化交易所更安全。

<img src="https://raw.githubusercontent.com/cosmos/cosmos/master/images/ex_network.png" height="300"/>

Cosmos Hub 不是宇宙的中心。任何区域都可以成为一个中心。