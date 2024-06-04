---
title: "Add Your First Query"
order: 6
description: Where you make it possible to retrieve a game
tags:
  - guided-coding
  - cosmos-sdk
---

# Add Your First Query

In the [previous section](./4-add-message.md) you added a message that lets you create new games. However, other than dumping the full storage, you cannot retrieve them yet as there is no query type defined. This section fixes that. You will:

* Add a query type to retrieve a game by index.
* Add all the necessary keeper and message server functions.
* Add CLI commands.

Note that you will only create a query to retrieve a single game, not a list of games, which would also be convenient. This other query is more complex and requires pagination.

## Add a game retrieval query

With the game storage indexed by game index, it is only natural to retrieve games by index. It is time to add a query to let anyone fetch games. You are going to:

* Create the query type.
* Compile Protobuf
* Create a query server next to the keeper.
* Handle the query.
* Register the necessary elements in the module.

### The game retrieval object type and Protobuf service

These are defined together in a new file `proto/alice/checkers/v1/query.proto` that you create:

```protobuf [https://github.com/b9lab/checkers-minimal/blob/query-game/proto/alice/checkers/v1/query.proto]
syntax = "proto3";
package alice.checkers.v1;

option go_package = "github.com/alice/checkers";

import "alice/checkers/v1/types.proto";
import "google/api/annotations.proto";
import "cosmos/query/v1/query.proto";
import "gogoproto/gogo.proto";

// Query defines the module Query service.
service Query {
  // GetGame returns the game at the requested index.
  rpc GetGame(QueryGetGameRequest) returns (QueryGetGameResponse) {
    option (cosmos.query.v1.module_query_safe) = true;
    option (google.api.http).get =
      "/alice/checkers/v1/game/{index}";
  }
}

// QueryGetGameRequest is the request type for the Query/GetGame RPC
// method.
message QueryGetGameRequest {
  // index defines the index of the game to retrieve.
  string index = 1;
}

// QueryGetGameResponse is the response type for the Query/GetGame RPC
// method.
message QueryGetGameResponse {
  // Game defines the game at the requested index.
  StoredGame Game = 1;
}
```

Note that the `StoredGame` included in the response does not need to include the `Index` field as it is implicit over the request's lifecycle.

### Compile Protobuf

Since you have defined a new query type and an associated `service`, you should recompile everything:

```sh
$ make proto-gen
```

In the new [`query.pg.go`](https://github.com/b9lab/checkers-minimal/blob/query-game/query.pb.go), you can find [`type QueryGetGameRequest struct`](https://github.com/b9lab/checkers-minimal/blob/query-game/query.pb.go#L35-L38) and [`type QueryServer interface`](https://github.com/b9lab/checkers-minimal/blob/query-game/query.pb.go#L191-L194). Now you can use them closer to the keeper.

### New query server

Create a new file `keeper/query_server.go` and take inspiration from `minimal-module-example`. It needs to:

* Implement the `GetGame` function.
* Return the game if it is found.
* Return no games and no errors if it is not found.
* Return an error otherwise.

```go [https://github.com/b9lab/checkers-minimal/blob/query-game/keeper/query_server.go]
package keeper

import (
    "context"
    "errors"

    "cosmossdk.io/collections"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"

    "github.com/alice/checkers"
)

var _ checkers.QueryServer = queryServer{}

// NewQueryServerImpl returns an implementation of the module QueryServer.
func NewQueryServerImpl(k Keeper) checkers.QueryServer {
    return queryServer{k}
}

type queryServer struct {
    k Keeper
}

// GetGame defines the handler for the Query/GetGame RPC method.
func (qs queryServer) GetGame(ctx context.Context, req *checkers.QueryGetGameRequest) (*checkers.QueryGetGameResponse, error) {
    game, err := qs.k.StoredGames.Get(ctx, req.Index)
    if err == nil {
        return &checkers.QueryGetGameResponse{Game: &game}, nil
    }
    if errors.Is(err, collections.ErrNotFound) {
        return &checkers.QueryGetGameResponse{Game: nil}, nil
    }

    return nil, status.Error(codes.Internal, err.Error())
}
```

Note how the response's `Game` is a pointer. This lets you define it as `nil` in the case there is no game at the requested index.

With the query server defined, you now need to register it in the module.

### Register the types in the module

Now that you have message types and server, you should register the service in `module/module.go`. Inspire yourself from what you find in `minimal-module-example`. The lines were previously commented out:

```diff-go [https://github.com/b9lab/checkers-minimal/blob/query-game/module/module.go]
    import (
+      "context"
        "encoding/json"
    )
    ...
    func (AppModule) RegisterGRPCGatewayRoutes(clientCtx client.Context, mux *gwruntime.ServeMux) {
-      // if err := checkers.RegisterQueryHandlerClient(context.Background(), mux, checkers.NewQueryClient(clientCtx)); err != nil {
-      //     panic(err)
-      // }
+      if err := checkers.RegisterQueryHandlerClient(context.Background(), mux, checkers.NewQueryClient(clientCtx)); err != nil {
+          panic(err)
+      }
    }
    ...
    func (am AppModule) RegisterServices(cfg module.Configurator) {
        // Register servers
        checkers.RegisterMsgServer(cfg.MsgServer(), keeper.NewMsgServerImpl(am.keeper))
-      // checkers.RegisterQueryServer(cfg.QueryServer(), keeper.NewQueryServerImpl(am.keeper))
+      checkers.RegisterQueryServer(cfg.QueryServer(), keeper.NewQueryServerImpl(am.keeper))
        ...
    }
    ...
```

### Add the CLI commands

At this stage, your module is able to handle queries passed to it by the app. But you are not yet able to craft such a query in the first place. When working from the command line, that crafting is handled by the CLI client. The CLI client usually tells you what queries it can create when you run `minid query --help`. Go ahead and check:

```sh
$ make install
$ minid query --help
```

You can see that `checkers` is missing from the list of available commands. Fix that by entering your desired command in `module/autocli.go`. Taking inspiration from `minimal-module-example`:

```diff-go [https://github.com/b9lab/checkers-minimal/blob/query-game/module/autocli.go#L11-L23]
    ...
-  Query: nil,
+  Query: &autocliv1.ServiceCommandDescriptor{
+      Service: checkersv1.Query_ServiceDesc.ServiceName,
+      RpcCommandOptions: []*autocliv1.RpcCommandOptions{
+          {
+              RpcMethod: "GetGame",
+              Use:       "get-game index",
+              Short:     "Get the current value of the game at index",
+              PositionalArgs: []*autocliv1.PositionalArgDescriptor{
+                  {ProtoField: "index"},
+              },
+          },
+      },
+  },
    ...
```

Note that the `get-game` in `"get-game index"` is parsed out and used as the command in the `minid query checkers get-game` command-line.

## Test again

Back in the minimal chain, compile to see the command that was added:

```sh
$ make install
```

Now, you should see the `get-game` command:

<CodeGroup>
<CodeGroupItem title="query">

```sh
$ minid query --help
```

Lists:

```txt
...
checkers            Querying commands for the checkers module
...
```

</CodeGroupItem>
<CodeGroupItem title="checkers">

```sh
$ minid query checkers --help
```

This returns:

```txt
...
Available Commands:
  get-game    Get the current value of the game at index
...
```

</CodeGroupItem>
<CodeGroupItem title="get-game">

```sh
$ minid query checkers get-game --help
```

This returns:

```txt
Get the current value of the game for an index

Usage:
  minid query checkers get-game index [flags]

Flags:
...
```

</CodeGroupItem>
</CodeGroup>

---

This time, adding a query did not change your genesis or storage, so you do not need to re-initialize. Start it:

```sh
$ minid start
```

Now query your previously created game:

```sh
$ minid query checkers get-game id1
```

This returns:

```txt
Game:
  black: mini16ajnus3hhpcsfqem55m5awf3mfwfvhpp36rc7d
  board: '*b*b*b*b|b*b*b*b*|*b*b*b*b|********|********|r*r*r*r*|*r*r*r*r|r*r*r*r*'
  index: id1
  red: mini1hv85y6h5rkqxgshcyzpn2zralmmcgnqwsjn3qg
  turn: b
```

Try to get a non-existent game:

```sh
$ minid query checkers get-game id2
```

This should return:

```txt
{}
```

Confirm this returned without any error:

```sh
$ echo $?
```

This returns:

```txt
0
```

## Conclusion

You have started creating the first elements of a simple checkers blockchain, one where it is possible to create, and recall, games.

Compared to earlier versions, Cosmos SDK v0.50 allows you to create modules in a more modular and concise way.

If you still want to reduce the amount of work to do, in a way reminiscent of _on Rails_ environments, you can use Ignite CLI. This is what you can learn in the next sections, where Ignite CLI is used to quickly create a checkers blockchain at an earlier (0.45) version of the Cosmos SDK.
