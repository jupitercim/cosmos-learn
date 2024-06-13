---
description: 构建您的第一个区块链和第一个 Cosmos SDK 查询。
title: 你好世界
---

# 使用 Ignite CLI 创建 "Hello world!" 区块链教程

**简介**

在本教程中，您将使用 Ignite CLI 构建一个简单的区块链，该区块链响应一个自定义查询，返回 "Hello %s!"，其中 "%s" 是查询中传递的名字。通过这个过程，您将增强对在 Cosmos SDK 区块链中创建自定义查询的理解。

## 设置和脚手架

1. **创建一个新的区块链：**

```bash
ignite scaffold chain hello
```

2. **导航到区块链目录：**

```bash
cd hello
```

## 添加自定义查询

- **生成查询：**

```bash
ignite scaffold query say-hello name --response name
```

此命令将为新查询 `say-hello` 生成代码，该查询接受一个名字作为输入，并在响应中返回它。

- **理解生成的代码：**

	- `proto/hello/hello/query.proto`：定义请求和响应结构。
	- `x/hello/client/cli/query_say_hello.go`：包含查询的 CLI 命令。
	- `x/hello/keeper/query_say_hello.go`：包含查询响应的逻辑。

## 自定义查询响应

在 Cosmos SDK 中，查询是从区块链请求信息的操作，用于访问如账本当前状态或交易详情等数据。虽然 SDK 提供了几种内置查询方法，但开发者也可以创建自定义查询，以实现特定数据检索或复杂操作。

- **修改 `query_say_hello.go`：**

更新 `x/hello/keeper/query_say_hello.go` 中的 `SayHello` 函数，以返回个性化的问候。

```go title="x/hello/keeper/query_say_hello.go"
package keeper

import (
	"context"
	"fmt"

	"hello/x/hello/types"

	sdk "github.com/cosmos/cosmos-sdk/types"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
)

func (q queryServer) SayHello(ctx context.Context, req *types.QuerySayHelloRequest) (*types.QuerySayHelloResponse, error) {
	if req == nil {
		return nil, status.Error(codes.InvalidArgument, "invalid request")
	}

	// 验证和上下文解包
	sdkCtx := sdk.UnwrapSDKContext(ctx)

	_ = sdkCtx
	// 自定义响应
	return &types.QuerySayHelloResponse{Name: fmt.Sprintf("Hello %s!", req.Name)}, nil
}
```

## 运行区块链

1. **启动区块链：**

```bash
ignite chain serve
```

2. **测试查询：**

使用命令行界面提交查询。

```
hellod q hello say-hello world
```

预期响应：`Hello world!`

## 结论

恭喜您！🎉 您已成功使用 Ignite CLI 创建了一个具有自定义查询的区块链模块。通过本教程，您学习了如何生成链、添加自定义查询以及修改逻辑以实现个性化响应。这展示了 Ignite CLI 在简化区块链开发中的强大功能，以及理解底层代码以实现自定义的重要性。