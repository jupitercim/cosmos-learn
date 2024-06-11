# hello

**hello** is a blockchain built using Cosmos SDK and Tendermint and created with [Ignite CLI](https://ignite.com/cli).

## Get started

```sh
ignite chain serve
```

`serve` command installs dependencies, builds, initializes, and starts your blockchain in development.

### Configure

Your blockchain in development can be configured with `config.yml`. To learn more, see
the [Ignite CLI docs](https://docs.ignite.com).

### Web Frontend

Additionally, Ignite CLI offers both Vue and React options for frontend scaffolding:

For a Vue frontend, use: `ignite scaffold vue`
For a React frontend, use: `ignite scaffold react`
These commands can be run within your scaffolded blockchain project.

For more information see the [monorepo for Ignite front-end development](https://github.com/ignite/web).

## Release

To release a new version of your blockchain, create and push a new tag with `v` prefix. A new draft release with the
configured targets will be created.

```
git tag v0.1
git push origin v0.1
```

After a draft release is created, make your final changes from the release page and publish it.

### Install

To install the latest version of your blockchain node's binary, execute the following command on your machine:

```
curl https://get.ignite.com/username/hello@latest! | sudo bash
```

`username/hello` should match the `username` and `repo_name` of the Github repository to which the source code was
pushed. Learn more about [the install process](https://github.com/allinbits/starport-installer).

## Learn more

该app/目录包含将区块链的不同部分连接在一起的文件。此目录中最重要的文件是app.go，其中包括区块链的类型定义以及用于创建和初始化区块链的函数。此文件负责将区块链的各个组件连接在一起并定义它们如何相互作用。

该cmd/目录包含负责编译二进制文件的命令行界面 (CLI) 的主包。此包定义了可以从 CLI
运行的命令以及应如何执行这些命令。它是区块链项目的重要组成部分，因为它为开发人员和用户提供了一种与区块链交互并执行各种任务的方式，例如查询区块链状态或发送交易。

该docs/目录用于存储项目文档。默认情况下，此目录包含一个 OpenAPI 规范文件，该文件是用于定义软件项目 API 的机器可读格式。OpenAPI
规范可用于自动生成项目的人类可读文档，并为其他工具和服务提供与 API 交互的方式。该docs/目录可用于存储与项目相关的任何其他文档。

该proto/目录包含协议缓冲区文件，用于描述区块链的数据结构。协议缓冲区是一种与语言和平台无关的序列化结构化数据的机制，常用于开发分布式系统，例如区块链网络。目录中的协议缓冲区文件proto/定义了区块链使用的数据结构和消息，并用于生成可用于与区块链交互的各种编程语言的代码。在
Cosmos SDK 的上下文中，协议缓冲区文件用于定义区块链可以发送和接收的特定类型的数据，以及可用于访问区块链功能的特定 RPC 端点。

该testutil/目录包含用于测试的辅助函数。这些函数提供了一种方便的方法来执行编写区块链测试时所需的常见任务，例如创建测试帐户、生成交易和检查区块链的状态。通过使用目录中的辅助函数testutil/，开发人员可以更快、更高效地编写测试，并确保他们的测试全面有效。

该x/目录包含已添加到区块链的自定义 Cosmos SDK 模块。标准 Cosmos SDK 模块是预构建的组件，可为基于 Cosmos SDK
的区块链提供常见功能，例如对质押和治理的支持。另一方面，自定义模块是专门为区块链项目开发的模块，可提供特定于项目的功能。

该config.yml文件是一个配置文件，可用于在开发过程中自定义区块链。此文件包括控制区块链各个方面的设置，例如网络的
ID、账户余额和节点参数。