
# Go Relayer




[Go relayer](https://github.com/cosmos/relayer)是一个用Golang编写的中继器实现。它可以创建客户端、连接和通道，传递数据包，以及更新和升级客户端。

Go relayer旨在通过最少的手动配置让你的中继器快速运行，并抽象掉许多更复杂的跨区块链通信协议（IBC）概念。其目标是让用户能够自行启动中继器并传递数据包。为了提供此功能，它自动化了大量工作，从[链注册表](https://github.com/cosmos/chain-registry)获取配置信息。

安装完成后，你将开始在主网链上中继并进行一些本地测试。

## 安装和开始

这个仓库提供了一个启动两条链的脚本，你需要它来测试中继器。

<!-- TODO: 一旦本地测试更新后，修改此文本 -->

<HighlightBox type="note">

Go relayer的最新主版本是v2。这个版本带来了重大更新和改进，包括引入了一个提供者接口，以适应具有不同共识类型的链（如CometBFT）和事件驱动的处理，从而提高了性能。
<br/><br/>
建议使用这个最新版本，以下命令假设你使用的是v2。

</HighlightBox>

1. 首先，为本节创建一个文件夹：

  ```sh
  $ mkdir relay-go-test
  $ cd relay-go-test
  ```

2. 克隆[Go relayer仓库](https://github.com/cosmos/relayer)：

  ```sh
  $ git clone https://github.com/cosmos/relayer.git
  ```

3. 现在构建Go relayer：

  ```sh
  $ cd relayer
  $ git checkout v2.4.2
  $ make install
  ```

确保你的`$GOPATH`设置正确，并且`$GOPATH/bin`包含在你的`$PATH`中。

要检查可用的命令，请运行`rly`二进制文件的帮助命令。

```sh
$ rly -h
```

输出如下：

```txt
rly has:
   1. 链和路径的配置管理
   2. 多链多密钥的密钥管理
   3. IBC的查询和交易功能

   注意：大多数命令都有别名，使其输入更快捷
         (例如 'rly tx', 'rly q', 等)

用法:
  rly [命令]

可用命令:
  config      管理配置文件
  chains      管理链配置
  paths       管理路径配置
  keys        管理中继器为每条链持有的密钥

  transact    IBC交易命令
  query       IBC查询命令
  start       在给定路径上启动监听中继器

  version     打印中继器版本信息
  help        帮助关于任何命令
  completion  为指定的shell生成自动完成脚本

标志:
  -d, --debug               调试输出
  -h, --help                rly的帮助
      --home string         设置主目录 (默认 "/Users/<userName>/.relayer")
      --log-format string   日志输出格式 (auto, logfmt, json, or console) (默认 "auto")

使用 "rly [命令] --help" 获取关于某个命令的更多信息。
```

注意类别如何反映你在上一节中看到的要求：管理链和路径信息，管理密钥，查询和交易。配置信息被添加到配置文件中，默认情况下存储在`$HOME/.relayer/config/config.yaml`。如果这是你第一次运行中继器，请首先使用以下命令初始化配置：

```sh
$ rly config init
```

并检查配置：

```sh
$ rly config show
```

<HighlightBox type="info">

默认情况下，交易将以`rly(VERSION)`的备忘录进行传递——例如`rly(v2.0.0)`。
<br/><br/>
要为所有中继定制备忘录，可以在初始化配置时使用`--memo`标志。

```shell
$ rly config init --memo "我的自定义备忘录"
```

自定义备忘录将附加`rly(VERSION)`。例如，一个备忘录为`我的自定义备忘录`且中继器版本为`v2.0.0`的交易备忘录将为`我的自定义备忘录 | rly(v2.0.0)`。
<br/><br/>
`--memo`标志也可用于涉及发送交易的其他`rly`命令，如`rly tx link`和`rly start`。如果需要，可以在这些命令中传递以覆盖`config.yaml`值。
<br/><br/>
要完全省略备忘录，包括默认值`rly(VERSION)`，请使用`-`作为备忘录。

</HighlightBox>

现在你已准备好添加你想要中继的链和路径，添加你的密钥并开始中继。你将研究两种不同的场景：在主网之间设置中继器（这将使你能够在生产中开始中继），以及设置两个本地链以进行测试。

## 生产中的中继

如前所述，Go relayer旨在让你的中继器在短时间内启动并运行。你将按照[Github仓库](https://github.com/cosmos/relayer)中的教程，在Cosmos Hub和Osmosis之间开始中继，这是最受欢迎的路径之一。

1. 配置你要中继的链。

在这个例子中，你将配置中继器在Cosmos Hub和Osmosis之间的规范路径上操作。

`rly chains add`命令从[链注册表](https://github.com/cosmos/chain-registry)获取链的元数据，并将其添加到你的配置文件中：

  ```sh
  $ rly chains add cosmoshub osmosis
  ```

从链注册表添加链会随机选择注册表条目中的一个公开可用的RPC地址。如果你正在运行自己的节点（如果你专业地运行中继服务，这是推荐的做法），请手动进入配置并将`rpc-addr`设置调整为你已暴露的RPC端点。

  <HighlightBox type="note">

`rly chains add`将检查链注册表中可用RPC端点的存活性。如果这些RPC端点都不可用，命令可能会失败。在这种情况下，你需要手动添加链配置。

  </HighlightBox>

要手动添加链配置文件，示例配置文件已包含在[Cosmos中继器文档](https://github.com/cosmos/relayer/tree/main/docs/example-configs/)中。

  ```sh
  $ rly chains add --url https://raw.githubusercontent.com/cosmos/relayer/main/docs/example-configs/cosmoshub-4.json
  $ rly chains add --url https://raw.githubusercontent.com/cosmos/relayer/main/docs/example-configs/osmosis-1.json
  ```

2. 导入或创建新的密钥供中继器在签名和中继交易时使用。

  <HighlightBox type="info">

`key-name`是你选择的标识符。

  </HighlightBox>

* 如果你需要生成一个新的私钥，你可以使用`add`子命令：

  ```sh
  $ rly keys add cosmoshub [key-name]
  $ rly keys add osmosis [key-name]
  ```

* 如果你已经有一个私钥并希望从助记词恢复它，你可以使用`restore`子命令：

  ```sh
  $ rly keys restore cosmoshub [key-name] "助记词"
  $ rly keys restore osmosis [key-name] "助记词"
  ```

3. 编辑中继器配置文件中的`key`值以匹配上面选择的`key-name`。

  <HighlightBox type="info">

如果你选择了`key-name`而不是“default”，此步骤是必要的。

  </HighlightBox>

示例：

   ```yaml
   - type: cosmos
     value:
     key: YOUR-KEY-NAME-HERE
     chain-id: cosmoshub-4
     rpc-addr: http://localhost:26657
   ```

4. 确保与配置链关联的密钥已获得资金。

<HighlightBox type="best-pr

actice">

为了让你的中继器正常运行，建议提供足够的资金，以防止由于链中费用的突然变化而中断中继操作。

  </HighlightBox>

  <HighlightBox type="note">

你可以检查每个链上关联的账户地址：

  ```sh
  $ rly keys show cosmoshub [key-name]
  $ rly keys show osmosis [key-name]
  ```

  </HighlightBox>

5. 配置链之间的路径。

  <HighlightBox type="note">

你可以通过JSON文件或直接从链上拉取路径。建议直接从链上拉取路径，因为这确保了你使用的路径和通道是最新的。

  </HighlightBox>

从Cosmos Hub和Osmosis之间现有的规范路径创建路径配置：

  ```sh
  $ rly paths add cosmoshub osmosis <path_name> --src-conn connection-0 --dst-conn connection-0
  ```

6. 创建IBC客户端、连接和通道。

  <HighlightBox type="warning">

尽量避免创建新的客户端和连接，除非现有的路径无法使用。通常情况下，链已经具有现有的客户端和连接，创建新的可能导致不必要的资源消耗。

  </HighlightBox>

使用以下命令检查链的现有客户端、连接和通道：

  ```sh
  $ rly q clients cosmoshub
  $ rly q connections cosmoshub
  $ rly q channels cosmoshub
  ```

  <HighlightBox type="note">

如果你已经找到了现有的客户端、连接和通道，你可以跳过这个步骤。否则，使用以下命令创建它们。

  </HighlightBox>

  ```sh
  $ rly tx link <path_name>
  ```

7. 启动中继器。

  <HighlightBox type="info">

`rly start`命令将开始监听配置路径上的交易并自动中继它们。

  </HighlightBox>

  ```sh
  $ rly start <path_name>
  ```

---

<HighlightBox type="success">

你现在已经配置并启动了你的Go relayer。🎉

</HighlightBox>