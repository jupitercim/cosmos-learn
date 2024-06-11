
## Contents

* [Block Explorers](#block-explorers)
    * [Visual Block Explorers](#visual-block-explorers)
    * [Terminal Block Explorers](#terminal-block-explorers)
* [Wallets](#wallets)
* [Chain Registry](#chain-registry)
* [Validators](#validators)
* [Cosmos SDK Modules](#cosmos-sdk-modules)
* [Monitoring](#monitoring)
* [Indexers](#indexers)
* [IBC](#ibc)

## Core Components

* [Cosmos Hub](https://github.com/cosmos/gaia)
<!-- -->
* [Cosmos SDK](https://github.com/cosmos/cosmos-sdk/)
* [IBC Go](https://github.com/cosmos/ibc-go)
* [CometBFT](https://github.com/cometbft/cometbft)
* [CosmWasm](https://github.com/CosmWasm/cosmwasm)
* [CosmJS](https://github.com/cosmos/cosmjs)


## Documentation

* [Cosmos Developer Portal](https://tutorials.cosmos.network)
* [Interchain Developer Academy](https://ida.interchain.io/)
* [Cosmos SDK](https://docs.cosmos.network/)
* [IBC](https://ibc.cosmos.com/)
* [CometBFT](https://docs.cometbft.com/)
* [Cosmos Hub](https://hub.cosmos.network/)
* [CosmWasm](https://docs.cosmwasm.com/docs/1.0/)
* [Cosmology](https://cosmology.tech/learn)

## Block Explorers

* [ATOMScan](https://atomscan.com)
* [Mintscan](https://www.mintscan.io)
    * [Mintscan for Cosmos Hub Testnet](https://cosmoshub-testnet.mintscan.io/cosmoshub-testnet)
* [Big Dipper](https://bigdipper.live) - [Source](https://github.com/forbole/big-dipper-2.0-cosmos)
* [IOBScan](https://ibc.iobscan.io/)

* [NG Explorers](https://explorers.guru/)
* [Ping.pub](https://ping.pub) - [Source](https://github.com/ping-pub/explorer)
* [Stake ID](https://stake.id)

### Visual Block Explorers

View Inter-Blockchain Communication (IBC) transfer activity. The map traces IBC transactions between different blockchains (called zones in the Cosmos Hub) to display accurate aggregate information about the pulse of the entire Cosmos ecosystem.

* [Map of Zones](https://mapofzones.com/?period=24) - [Source](https://github.com/mapofzones)
* [Mintscan](https://hub.mintscan.io) - Interchain Explorer by Cosmostation.

### Terminal Block Explorers

Explore Cosmos SDK blockchains via a terminal.

* [gex](https://github.com/cosmos/gex) - GEX In-Terminal Explorer.
* [cshtop](https://github.com/gsk967/cshtop) - Cosmos htop , Blocks visualizer on terminal.
* [pvtop](https://github.com/blockpane/pvtop) - Consensus visualizer on terminal.
* [tmtop](https://github.com/quokkastake/tmtop) - Htop-like visualiser of consensus inspired by pvtop that allows showing upgrade info, working with consumer chains and non-Cosmos chains and way more.

## Chain Registry

A registry containing standardized metadata from most Cosmos chains.

* [cosmos/chain-registry](https://github.com/cosmos/chain-registry/)
* [Cosmos directory](https://cosmos.directory) - [Source](https://github.com/eco-stake/cosmos-directory)
* [cosmology-tech/chain-registry](https://github.com/cosmology-tech/chain-registry) - A npm package for the official Cosmos Chain Registry.

## Validators

Popular block explorers provide a list of active validators. The easiest entry point to view validator profiles is from a block explorer:

* [List on Mintscan](https://www.mintscan.io/cosmos/validators)
* [List on ATOMScan](https://atomscan.com/validators)
* [List on BigDipper](https://cosmos.bigdipper.live/validators)
* [List on Kujira POD](https://pod.kujira.app/cosmoshub-4)

DYOR when choosing a validator. Consider delegating your tokens to validators outside of the top 20 to increase the decentralization of the network.
This is also a good practice to avoid 0% commission validators and exchange validators.

## Cosmos SDK Modules

The best place to find an accurate list of the Cosmos SDK modules is the project repository:

* For a list of production-grade modules, see the [List of Modules](https://docs.cosmos.network/main/modules/).
* For a list of well-known third-party modules, see [Cosmod.xyz](https://cosmod.xyz)

## Monitoring

* [PANIC Monitoring and Alerting For Blockchains](https://github.com/SimplyVC/panic) - An open source monitoring and alerting solution for Cosmos SDK, Substrate, and Chainlink-based nodes.
* [Prometheus Exporter](https://github.com/node-a-team/Cosmos-IE) - An integrated Prometheus exporter for the Cosmos SDK.
* [Cosmos Chains Dashboard](https://github.com/zhangyelong/cosmos-dashboard) - A Grafana dashboard to monitor Cosmos SDK and Tendermint-based blockchain nodes.
* [Chain Pulse](https://github.com/informalsystems/chainpulse) - Relayed IBC packets monitor with Prometheus exporter.
* [missed-blocks-checker](https://github.com/QuokkaStake/missed-blocks-checker) - Monitor validators' missed blocks on multiple Cosmos chains and send its notifications to Telegram.
* [Nodes Checker](https://t.me/NodesGuru_bot) - Check your nodes status online, receive instant notification if something is wrong with your validator node.
* [Cosmon](https://github.com/iqlusioninc/cosmon) - Observability tool for Cosmos and other Tendermint applications.
* [Tenderduty](https://github.com/blockpane/tenderduty) - Comprehensive monitoring tool for Tendermint chains. Its primary function is to alert a validator if they are missing blocks, and more.
* [UpgradesWatchdog](https://github.com/ChihuahuaChain/UpgradesWatchdog) - SoftwareUpgradeProposal & GitHub Releases telegram monitoring tool.
* [cosmos-node-exporter](https://github.com/QuokkaStake/cosmos-node-exporter.git) - A Prometheus exporter to scrape data on your node sync status, Cosmovisor upgrades and GitHub version mismatches, useful for node operators and validators.
* [cosmos-wallets-exporter](https://github.com/QuokkaStake/cosmos-wallets-exporter.git) - A Prometheus exporter to scrape data on wallets balance, useful to get notified if your wallet balance is too low.
* [cosmos-validators-exporter](https://github.com/QuokkaStake/cosmos-validators-exporter.git) - A Prometheus exporter to scrape data about a validator (missed blocks, delegators count, total staked amount, rankings, etc.)
* [cosmos-proposals-checker](https://github.com/QuokkaStake/cosmos-proposals-checker.git) - A bot that sends you a notification on multiple Cosmos chains if your wallet hasn't voted on any proposal.
* [cosmos-transactions-bot](https://github.com/QuokkaStake/cosmos-transactions-bot.git) - A bot that sends you notifications on any transactions you want to be subscribed to on multiple Cosmos chains.

## Indexers

* [Cosmscan](https://github.com/cosmscan/cosmscan-go) - An indexer engine for Cosmos chains.
* [interchain-indexer](https://github.com/Reecepbcups/interchain-indexer) - A cosmos blockchain indexer in Python.
* [Cosmos Indexer](https://github.com/DefiantLabs/cosmos-indexer) - A generalized DB schema indexer with correlation and direct database indexing in Go.
* [BDJuno](https://github.com/forbole/bdjuno) - All the chains' data that are queried from the RPC and gRPC endpoints are stored inside a PostgreSQL database on top of which GraphQL APIs can then be created using Hasura.

## IBC

* [IBCprotocol.dev](https://ibcprotocol.dev/) - IBC Protocol website.
* [Interchain Standards](https://github.com/cosmos/ibc/) - Interchain Standards (ICS) for the Cosmos network & interchain ecosystem.
* [cosmos/ibc-go](https://github.com/cosmos/ibc-go) - Inter-Blockchain Communication protocol (IBC) implementation in Go.
* [cosmos/ibc-rs](https://github.com/cosmos/ibc-rs) - Rust implementation of the Inter-Blockchain Communication (IBC) protocol.
* [interchaintest](https://github.com/strangelove-ventures/interchaintest) - E2E testing framework for IBC Chains.
* [cosmos/relayer](https://github.com/cosmos/relayer) - IBC Relayer in Go.
* [informalsystems/hermes](https://github.com/informalsystems/hermes) - IBC Relayer in Rust.
* [confio/ts-relayer](https://github.com/confio/ts-relayer) - IBC Relayer in TypeScript.
* [local-interchain](https://github.com/Reecepbcups/local-interchain) - Quickly spin up a local IBC development environment on any operating system.



## Wallets

