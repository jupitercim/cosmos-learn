在 Cosmos SDK 中，自定义代币名称涉及到在链上创建新的代币并定义其元数据。这通常是在创建模块时完成的，例如 `bank` 模块。在这个模块中，可以通过编写自定义的模块配置和处理逻辑来定义新的代币名称。

以下是一个基本示例，演示如何在 Cosmos SDK 中自定义代币名称：

### 1. 定义新的代币

首先，您需要定义新的代币名称并在链上注册。

```go
package app

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/x/bank/types"
)

const (
    MyTokenDenom = "mytoken" // 自定义代币名称
)

func RegisterCustomCoin(app *App) {
    // 注册新的代币元数据
    app.BankKeeper.SetDenomMetaData(app.ctx, types.Metadata{
        Description: "My custom token",
        DenomUnits: []*types.DenomUnit{
            {Denom: "mytoken", Exponent: 0, Aliases: []string{"mtk"}},
        },
        Base:    "mytoken",
        Display: "mytoken",
    })
}
```

### 2. 在应用程序初始化时注册新的代币

在您的应用程序初始化代码中调用 `RegisterCustomCoin` 函数。

```go
func NewApp(...) *App {
    ...
    app := &App{
        ...
    }

    // 注册自定义代币
    RegisterCustomCoin(app)

    return app
}
```

### 3. 在交易中使用自定义代币

在构建交易时，您可以使用新的代币名称。例如，在发送交易时：

```go
msg := banktypes.NewMsgSend(
    fromAddress,
    toAddress,
    sdk.NewCoins(sdk.NewCoin("mytoken", sdk.NewInt(1000))),
)
```

### 4. 更新配置文件

确保在配置文件中包含自定义代币的相关信息。例如，更新 `app.toml` 或 `config.toml` 文件以反映新的代币设置。

### 5. 编译和部署链

完成上述步骤后，您需要重新编译和部署您的链。确保所有节点和客户端都更新到新的代码版本。

```sh
make build
```

### 6. 测试和验证

启动链并验证新的代币是否正确注册并可用。您可以使用 CLI 工具或前端应用程序测试新的代币功能。

```sh
./appd start
```

然后，使用 CLI 工具检查代币元数据：

```sh
./appd query bank denom-metadata mytoken
```

### 总结

通过上述步骤，您可以在 Cosmos SDK 中定义和注册自定义代币名称。这些步骤包括定义新的代币元数据、在应用程序初始化时注册代币、在交易中使用自定义代币，并确保配置文件正确更新。编译和部署新的链版本后，您可以测试和验证新的代币功能。