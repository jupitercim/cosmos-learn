---
title: "Build Your Module"
order: 3
description: Where you create your checkers module
tags:
  - guided-coding
  - cosmos-sdk
---

# Build Your Module

You will integrate your module with the minimal chain that you [cloned previously](./1-preparation.md). To facilitate integration and speed up feedback, you will direct the Go package manager to look for this dependency on a local disk. The simplest way is to put the module folder into a folder next to the minimal chain's folder:

```sh
$ mkdir checkers-minimal
```

The `ls` command should give you:

```txt
chain-minimal                   checkers-minimal                minimal-module-example
```

You can now define the module:

```sh
$ cd checkers-minimal
$ touch go.mod
```

In the new file, you define your module name and the Go version:

```txt
module github.com/alice/checkers

go 1.21
```

## Protobuf files

On top of the _regular_ Protobuf files, like message and query types, your checkers module type will also be identified with a Protobuf file. You have chosen the package `github.com/alice/checkers`, so go ahead and create the necessary folders:

```sh
$ mkdir -p proto/alice/checkers/v1
$ mkdir -p proto/alice/checkers/module/v1
```

### Module proto

You define your module in a new `proto/alice/checkers/module/v1/module.proto`, taking inspiration from `minimal-module-example`:

```protobuf [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/proto/alice/checkers/module/v1/module.proto]
syntax = "proto3";

package alice.checkers.module.v1;

import "cosmos/app/v1alpha1/module.proto";

// Module is the app config object of the module.
// Learn more: https://docs.cosmos.network/main/building-modules/depinject
message Module {
  option (cosmos.app.v1alpha1.module) = {
    go_import : "github.com/alice/checkers"
  };

  // authority defines the custom module authority.
  // if not set, defaults to the governance module.
  string authority = 1;
}
```

<HighlightBox type="note">

Note how the package is `alice.checkers.module.v1` and the message name is `Module`. This means that later on, when you integrate the module in your chain, you will identify it with:

```txt
alice.checkers.module.v1.Module
```

</HighlightBox>

### Necessary Protobuf files

The compilation of your Protobuf files will be done with the `scripts/protocgen.sh` script, which you can copy from `minimal-module-example`:

```sh
$ mkdir scripts
$ cp ../minimal-module-example/scripts/protocgen.sh ./scripts
```

<ExpansionPanel title="Content of copied file">

```sh [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/scripts/protocgen.sh]
#!/usr/bin/env bash

set -e

echo "Generating gogo proto code"
cd proto
proto_dirs=$(find . -path -prune -o -name '*.proto' -print0 | xargs -0 -n1 dirname | sort | uniq)
for dir in $proto_dirs; do
  for file in $(find "${dir}" -maxdepth 1 -name '*.proto'); do
    # this regex checks if a proto file has its go_package set to github.com/alice/checkers/api/...
    # gogo proto files SHOULD ONLY be generated if this is false
    # you don't want gogo proto to run for proto files which are natively built for google.golang.org/protobuf
    if grep -q "option go_package" "$file" && grep -H -o -c 'option go_package.*github.com/alice/checkers/api' "$file" | grep -q ':0$'; then
      buf generate --template buf.gen.gogo.yaml $file
    fi
  done
done

echo "Generating pulsar proto code"
buf generate --template buf.gen.pulsar.yaml

cd ..

cp -r github.com/alice/checkers/* ./
rm -rf api && mkdir api
mv alice/checkers/* ./api
rm -rf github.com alice
```

</ExpansionPanel>

Note how it mentions two files: `buf.gen.gogo.yaml` and `buf.gen.pulsar.yaml`. Go ahead and copy them from `minimal-module-example`. In fact, also copy `buf.yaml` and the version lock file to avoid surprises:

```sh
$ cp ../minimal-module-example/proto/buf* ./proto
```

<ExpansionPanel title="Content of copied files">

<CodeGroup>
<CodeGroupItem title="buf.gen.gogo.yaml">

```yaml [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/proto/buf.gen.gogo.yaml]
version: v1
plugins:
  - name: gocosmos
    out: ..
    opt: plugins=grpc,Mgoogle/protobuf/any.proto=github.com/cosmos/cosmos-sdk/codec/types,Mcosmos/orm/v1/orm.proto=cosmossdk.io/orm
  - name: grpc-gateway
    out: ..
    opt: logtostderr=true,allow_colon_final_segments=true
```

</CodeGroupItem>
<CodeGroupItem title="buf.gen.pulsar.yaml">

```yaml [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/proto/buf.gen.pulsar.yaml]
version: v1
managed:
  enabled: true
  go_package_prefix:
    default: github.com/alice/checkers/api
    except:
      - buf.build/googleapis/googleapis
      - buf.build/cosmos/gogo-proto
      - buf.build/cosmos/cosmos-proto
      - buf.build/cosmos/cosmos-sdk
plugins:
  - name: go-pulsar
    out: ..
    opt: paths=source_relative,Mcosmos/app/v1alpha1/module.proto=cosmossdk.io/api/cosmos/app/v1alpha1,Mcosmos/base/v1beta1/coin.proto=cosmossdk.io/api/cosmos/base/v1beta1,Mcosmos/base/query/v1beta1/pagination.proto=cosmossdk.io/api/cosmos/base/query/v1beta1
  - name: go-grpc
    out: ..
    opt: paths=source_relative,Mcosmos/app/v1alpha1/module.proto=cosmossdk.io/api/cosmos/app/v1alpha1
  - name: go-cosmos-orm
    out: ..
    opt: paths=source_relative,Mcosmos/app/v1alpha1/module.proto=cosmossdk.io/api/cosmos/app/v1alpha1
```

</CodeGroupItem>
<CodeGroupItem title="buf.lock">

```yaml [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/proto/buf.lock]
# Generated by buf. DO NOT EDIT.
version: v1
deps:
  - remote: buf.build
    owner: cosmos
    repository: cosmos-proto
    commit: 1935555c206d4afb9e94615dfd0fad31
    digest: shake256:c74d91a3ac7ae07d579e90eee33abf9b29664047ac8816500cf22c081fec0d72d62c89ce0bebafc1f6fec7aa5315be72606717740ca95007248425102c365377
  - remote: buf.build
    owner: cosmos
    repository: cosmos-sdk
    commit: cf13c7d232dd405180c2af616fa8a075
    digest: shake256:769a38e306a98339b549bc96991c97fae8bd3ceb1a7646c7bfe9a74e406ab068372970fbc5abda1891e2f3c36527cf2d3a25f631739d36900787226e564bb612
  - remote: buf.build
    owner: cosmos
    repository: gogo-proto
    commit: 5e5b9fdd01804356895f8f79a6f1ddc1
    digest: shake256:0b85da49e2e5f9ebc4806eae058e2f56096ff3b1c59d1fb7c190413dd15f45dd456f0b69ced9059341c80795d2b6c943de15b120a9e0308b499e43e4b5fc2952
  - remote: buf.build
    owner: googleapis
    repository: googleapis
    commit: 28151c0d0a1641bf938a7672c500e01d
    digest: shake256:49215edf8ef57f7863004539deff8834cfb2195113f0b890dd1f67815d9353e28e668019165b9d872395871eeafcbab3ccfdb2b5f11734d3cca95be9e8d139de
  - remote: buf.build
    owner: protocolbuffers
    repository: wellknowntypes
    commit: 657250e6a39648cbb169d079a60bd9ba
    digest: shake256:00de25001b8dd2e29d85fc4bcc3ede7aed886d76d67f5e0f7a9b320b90f871d3eb73507d50818d823a0512f3f8db77a11c043685528403e31ff3fef18323a9fb
```

</CodeGroupItem>
<CodeGroupItem title="buf.yaml">

```yaml [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/proto/buf.yaml]
version: v1
deps:
  - buf.build/cosmos/cosmos-sdk # pin the Cosmos SDK version
  - buf.build/cosmos/cosmos-proto
  - buf.build/cosmos/gogo-proto
  - buf.build/googleapis/googleapis
lint:
  use:
    - DEFAULT
    - COMMENTS
    - FILE_LOWER_SNAKE_CASE
  except:
    - UNARY_RPC
    - COMMENT_FIELD
    - SERVICE_SUFFIX
    - PACKAGE_VERSION_SUFFIX
    - RPC_REQUEST_STANDARD_NAME
```

</CodeGroupItem>
</CodeGroup>

</ExpansionPanel>

To make your life easier, `minimal-module-example` also provides a `make` target to compile all the Protobuf files. Copy the whole `Makefile`:

```sh
$ cp ../minimal-module-example/Makefile .
```

<ExpansionPanel title="Content of copied file">

```lang-makefile [https://github.com/b9lab/checkers-minimal/blob/initial-protobuf/Makefile]
#!/usr/bin/make -f

DOCKER := $(shell which docker)

#################
###   Build   ###
#################

test:
    @echo "--> Running tests"
    go test -v ./...

test-integration:
    @echo "--> Running integration tests"
    cd integration; go test -v ./...

.PHONY: test test-integration

##################
###  Protobuf  ###
##################

protoVer=0.14.0
protoImageName=ghcr.io/cosmos/proto-builder:$(protoVer)
protoImage=$(DOCKER) run --rm -v $(CURDIR):/workspace --workdir /workspace $(protoImageName)

proto-all: proto-format proto-lint proto-gen

proto-gen:
    @echo "Generating protobuf files..."
    @$(protoImage) sh ./scripts/protocgen.sh
    @go mod tidy

proto-format:
    @$(protoImage) find ./ -name "*.proto" -exec clang-format -i {} \;

proto-lint:
    @$(protoImage) buf lint proto/ --error-format=json

.PHONY: proto-all proto-gen proto-format proto-lint

#################
###  Linting  ###
#################

golangci_lint_cmd=golangci-lint
golangci_version=v1.51.2

lint:
    @echo "--> Running linter"
    @go install github.com/golangci/golangci-lint/cmd/golangci-lint@$(golangci_version)
    @$(golangci_lint_cmd) run ./... --timeout 15m

lint-fix:
    @echo "--> Running linter and fixing issues"
    @go install github.com/golangci/golangci-lint/cmd/golangci-lint@$(golangci_version)
    @$(golangci_lint_cmd) run ./... --fix --timeout 15m

.PHONY: lint lint-fix
```

</ExpansionPanel>

Note how it uses the [`ghcr.io/cosmos/proto-builder:0.14.0`](https://github.com/cosmos/cosmos-sdk/pkgs/container/proto-builder/119928846?tag=0.14.0) Docker image here to run the `protocgen.sh` script. This helps with making sure all necessary software is available.

### First Protobuf compilation

Before you run the compilation, because you have not yet defined any Protobuf files for messages, you can comment out this `protocgen.sh` line:

```diff-sh [https://github.com/b9lab/checkers-minimal/compare/initial-protobuf..prevent-error#diff-63113d53e02a4e334ffa1a1ba0c308329c7acd431153571a2c02ce9268af5816R24]
-   cp -r github.com/alice/checkers/* ./
+   # cp -r github.com/alice/checkers/* ./
```

Now run the compilation:

```sh
$ make proto-gen
```

This creates a new [`api/module/v1/module.pulsar.go`](https://github.com/b9lab/checkers-minimal/blob/first-proto-gen/api/module/v1/module.pulsar.go) file and updates your `go.mod`. It also creates a new `go.sum`.

Now you need a couple more Protobuf files.

### Minimum Protobuf objects

You are not defining your games, messages, and queries just yet. However, you already define the facts that your module:

* Has a genesis, which has a type.
* Uses params, which also have a type.

Go ahead and define them in a new `proto/alice/checkers/v1/types.proto`, taking inspiration from `minimal-module-example`:

```protobuf [https://github.com/b9lab/checkers-minimal/blob/first-types/proto/alice/checkers/v1/types.proto]
syntax = "proto3";
package alice.checkers.v1;

option go_package = "github.com/alice/checkers";

import "cosmos_proto/cosmos.proto";
import "gogoproto/gogo.proto";

// Params defines the parameters of the module.
message Params {}

// GenesisState is the state that must be provided at genesis.
message GenesisState {
  // params defines all the parameters of the module.
  Params params = 1 [ (gogoproto.nullable) = false ];
}
```

Because you added a new Protobuf file, you need to compile it too. Uncomment the `cp` line in `protocgen.sh`:

```diff-sh
-   # cp -r github.com/alice/checkers/* ./
+   cp -r github.com/alice/checkers/* ./
```

And run it:

```sh
$ make proto-gen
```

<HighlightBox type="note" title="Troubleshooting">

If the process tells you that it downloads a Cosmos SDK version different from 0.50.1, make sure to manually edit your `go.mod` after the fact.

```diff [https://github.com/b9lab/checkers-minimal/blob/root-files/go.mod#L8]
-       github.com/cosmos/cosmos-sdk v0.47.5
+       github.com/cosmos/cosmos-sdk v0.50.1
```

Next, tidy up the dependencies:

```sh
$ go mod tidy
```

</HighlightBox>

The script has created two new files: `api/v1/types.pulsar.go` and `types.pb.go`.

Your module is not viable yet. You need to define it and at least have it conform to the interface expected by an app, in this case `chain-minimal`.

## Module interface

Once again, you can take inspiration from `minimal-module-example`.

### General files

Prepare the basics around codec, params, genesis, and module name. You can keep them in the root folder:

<CodeGroup>
<CodeGroupItem title="codec.go">

```go [https://github.com/b9lab/checkers-minimal/blob/root-files/codec.go]
package checkers

import (
    types "github.com/cosmos/cosmos-sdk/codec/types"
)

// RegisterInterfaces registers the interfaces types with the interface registry.
func RegisterInterfaces(registry types.InterfaceRegistry) {
}
```

You do not have message and query definitions, so you can keep the function body empty.

</CodeGroupItem>
<CodeGroupItem title="params.go">

```go [https://github.com/b9lab/checkers-minimal/blob/root-files/params.go]
package checkers

// DefaultParams returns default module parameters.
func DefaultParams() Params {
    return Params{
        // Set default values here.
    }
}

// Validate does the sanity check on the params.
func (p Params) Validate() error {
    // Sanity check goes here.
    return nil
}
```

There is nothing to define except the default functions.

</CodeGroupItem>
<CodeGroupItem title="genesis.go">

```go [https://github.com/b9lab/checkers-minimal/blob/root-files/genesis.go]
package checkers

// NewGenesisState creates a new genesis state with default values.
func NewGenesisState() *GenesisState {
    return &GenesisState{
        Params: DefaultParams(),
    }
}

// Validate performs basic genesis state validation returning an error upon any
func (gs *GenesisState) Validate() error {
    if err := gs.Params.Validate(); err != nil {
        return err
    }

    return nil
}
```

You have not yet defined any storage, other than `Params`, so this is simple.

</CodeGroupItem>
<CodeGroupItem title="keys.go">

```go [https://github.com/b9lab/checkers-minimal/blob/root-files/keys.go]
package checkers

import "cosmossdk.io/collections"

const ModuleName = "checkers"

var (
    ParamsKey  = collections.NewPrefix("Params")
)
```

For now, all you need is the module's name and the storage key at which you can find the params.

</CodeGroupItem>
</CodeGroup>

---

The files have a lot of missing dependencies, so go ahead and, once more, run:

```sh
$ go mod tidy
```

See the previous troubleshoot if it updates to a `cosmos-sdk` other than `v0.50.1`.

With this done, you can move to defining some necessary functions of the module's keeper.

### Keeper files

Again `minimal-module-example` is a good guide. Create the keeper folder:

```sh
$ mkdir keeper
```

In it, you define what the keeper holds, its genesis actions, and eventually its message and query servers:

<CodeGroup>
<CodeGroupItem title="keeper.go">

```go [https://github.com/b9lab/checkers-minimal/blob/keeper-files/keeper/keeper.go]
package keeper

import (
    "fmt"

    "cosmossdk.io/collections"
    "cosmossdk.io/core/address"
    storetypes "cosmossdk.io/core/store"
    "github.com/cosmos/cosmos-sdk/codec"

    "github.com/alice/checkers"
)

type Keeper struct {
    cdc          codec.BinaryCodec
    addressCodec address.Codec

    // authority is the address capable of executing a MsgUpdateParams and other authority-gated message.
    // typically, this should be the x/gov module account.
    authority string

    // state management
    Schema collections.Schema
    Params collections.Item[checkers.Params]
}

// NewKeeper creates a new Keeper instance
func NewKeeper(cdc codec.BinaryCodec, addressCodec address.Codec, storeService storetypes.KVStoreService, authority string) Keeper {
    if _, err := addressCodec.StringToBytes(authority); err != nil {
        panic(fmt.Errorf("invalid authority address: %w", err))
    }

    sb := collections.NewSchemaBuilder(storeService)
    k := Keeper{
        cdc:          cdc,
        addressCodec: addressCodec,
        authority:    authority,
        Params:       collections.NewItem(sb, checkers.ParamsKey, "params", codec.CollValue[checkers.Params](cdc)),
    }

    schema, err := sb.Build()
    if err != nil {
        panic(err)
    }

    k.Schema = schema

    return k
}

// GetAuthority returns the module's authority.
func (k Keeper) GetAuthority() string {
    return k.authority
}
```

With this one, it helps to have a model to copy from.

</CodeGroupItem>
<CodeGroupItem title="genesis.go">

```go [https://github.com/b9lab/checkers-minimal/blob/keeper-files/keeper/genesis.go]
package keeper

import (
    "context"

    "github.com/alice/checkers"
)

// InitGenesis initializes the module state from a genesis state.
func (k *Keeper) InitGenesis(ctx context.Context, data *checkers.GenesisState) error {
    if err := k.Params.Set(ctx, data.Params); err != nil {
        return err
    }

    return nil
}

// ExportGenesis exports the module state to a genesis state.
func (k *Keeper) ExportGenesis(ctx context.Context) (*checkers.GenesisState, error) {
    params, err := k.Params.Get(ctx)
    if err != nil {
        return nil, err
    }

    return &checkers.GenesisState{
        Params: params,
    }, nil
}
```

</CodeGroupItem>
</CodeGroup>

---

You may need to run `go mod tidy` again.

At the moment you have not defined any message or query types, so you do not bother with defining the corresponding servers.

### Module files

To be correctly called, the module needs to conform to the `chain-minimal` expectations. You define functions for the module, the dependency injection, and accessorily to open the possibility to add CLI commands. Create the `module` folder:

```sh
$ mkdir module
```

In it, again copying from `minimal-module-example`:

<CodeGroup>
<CodeGroupItem title="module.go">

```go [https://github.com/b9lab/checkers-minimal/blob/module-files/module/module.go]
package module

import (
    "encoding/json"
    "fmt"

    "cosmossdk.io/core/appmodule"
    gwruntime "github.com/grpc-ecosystem/grpc-gateway/runtime"

    "github.com/cosmos/cosmos-sdk/client"
    "github.com/cosmos/cosmos-sdk/codec"
    codectypes "github.com/cosmos/cosmos-sdk/codec/types"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/types/module"

    "github.com/alice/checkers"
    "github.com/alice/checkers/keeper"
)

var (
    _ module.AppModuleBasic = AppModule{}
    _ module.HasGenesis     = AppModule{}
    _ appmodule.AppModule   = AppModule{}
)

// ConsensusVersion defines the current module consensus version.
const ConsensusVersion = 1

type AppModule struct {
    cdc    codec.Codec
    keeper keeper.Keeper
}

// NewAppModule creates a new AppModule object
func NewAppModule(cdc codec.Codec, keeper keeper.Keeper) AppModule {
    return AppModule{
        cdc:    cdc,
        keeper: keeper,
    }
}

func NewAppModuleBasic(m AppModule) module.AppModuleBasic {
    return module.CoreAppModuleBasicAdaptor(m.Name(), m)
}

// Name returns the checkers module's name.
func (AppModule) Name() string { return checkers.ModuleName }

// RegisterLegacyAminoCodec registers the checkers module's types on the LegacyAmino codec.
// New modules do not need to support Amino.
func (AppModule) RegisterLegacyAminoCodec(cdc *codec.LegacyAmino) {}

// RegisterGRPCGatewayRoutes registers the gRPC Gateway routes for the checkers module.
func (AppModule) RegisterGRPCGatewayRoutes(clientCtx client.Context, mux *gwruntime.ServeMux) {
    // if err := checkers.RegisterQueryHandlerClient(context.Background(), mux, checkers.NewQueryClient(clientCtx)); err != nil {
    //     panic(err)
    // }
}

// RegisterInterfaces registers interfaces and implementations of the checkers module.
func (AppModule) RegisterInterfaces(registry codectypes.InterfaceRegistry) {
    checkers.RegisterInterfaces(registry)
}

// ConsensusVersion implements AppModule/ConsensusVersion.
func (AppModule) ConsensusVersion() uint64 { return ConsensusVersion }

// RegisterServices registers a gRPC query service to respond to the module-specific gRPC queries.
func (am AppModule) RegisterServices(cfg module.Configurator) {
    // Register servers
    // checkers.RegisterMsgServer(cfg.MsgServer(), keeper.NewMsgServerImpl(am.keeper))
    // checkers.RegisterQueryServer(cfg.QueryServer(), keeper.NewQueryServerImpl(am.keeper))

    // Register in place module state migration migrations
    // m := keeper.NewMigrator(am.keeper)
    // if err := cfg.RegisterMigration(checkers.ModuleName, 1, m.Migrate1to2); err != nil {
    //     panic(fmt.Sprintf("failed to migrate x/%s from version 1 to 2: %v", checkers.ModuleName, err))
    // }
}

// DefaultGenesis returns default genesis state as raw bytes for the module.
func (AppModule) DefaultGenesis(cdc codec.JSONCodec) json.RawMessage {
    return cdc.MustMarshalJSON(checkers.NewGenesisState())
}

// ValidateGenesis performs genesis state validation for the circuit module.
func (AppModule) ValidateGenesis(cdc codec.JSONCodec, _ client.TxEncodingConfig, bz json.RawMessage) error {
    var data checkers.GenesisState
    if err := cdc.UnmarshalJSON(bz, &data); err != nil {
        return fmt.Errorf("failed to unmarshal %s genesis state: %w", checkers.ModuleName, err)
    }

    return data.Validate()
}

// InitGenesis performs genesis initialization for the checkers module.
// It returns no validator updates.
func (am AppModule) InitGenesis(ctx sdk.Context, cdc codec.JSONCodec, data json.RawMessage) {
    var genesisState checkers.GenesisState
    cdc.MustUnmarshalJSON(data, &genesisState)

    if err := am.keeper.InitGenesis(ctx, &genesisState); err != nil {
        panic(fmt.Sprintf("failed to initialize %s genesis state: %v", checkers.ModuleName, err))
    }
}

// ExportGenesis returns the exported genesis state as raw bytes for the circuit
// module.
func (am AppModule) ExportGenesis(ctx sdk.Context, cdc codec.JSONCodec) json.RawMessage {
    gs, err := am.keeper.ExportGenesis(ctx)
    if err != nil {
        panic(fmt.Sprintf("failed to export %s genesis state: %v", checkers.ModuleName, err))
    }

    return cdc.MustMarshalJSON(gs)
}
```

Note how you comment out the lines that will eventually register the missing message and query servers and client.

</CodeGroupItem>
<CodeGroupItem title="depinject.go">

```go [https://github.com/b9lab/checkers-minimal/blob/module-files/module/depinject.go]
package module

import (
    "cosmossdk.io/core/address"
    "cosmossdk.io/core/appmodule"
    "cosmossdk.io/core/store"
    "cosmossdk.io/depinject"

    "github.com/cosmos/cosmos-sdk/codec"
    authtypes "github.com/cosmos/cosmos-sdk/x/auth/types"

    modulev1 "github.com/alice/checkers/api/module/v1"
    "github.com/alice/checkers/keeper"
)

var _ appmodule.AppModule = AppModule{}

// IsOnePerModuleType implements the depinject.OnePerModuleType interface.
func (am AppModule) IsOnePerModuleType() {}

// IsAppModule implements the appmodule.AppModule interface.
func (am AppModule) IsAppModule() {}

func init() {
    appmodule.Register(
        &modulev1.Module{},
        appmodule.Provide(ProvideModule),
    )
}

type ModuleInputs struct {
    depinject.In

    Cdc          codec.Codec
    StoreService store.KVStoreService
    AddressCodec address.Codec

    Config *modulev1.Module
}

type ModuleOutputs struct {
    depinject.Out

    Module appmodule.AppModule
    Keeper keeper.Keeper
}

func ProvideModule(in ModuleInputs) ModuleOutputs {
    // default to governance as authority if not provided
    authority := authtypes.NewModuleAddress("gov")
    if in.Config.Authority != "" {
        authority = authtypes.NewModuleAddressOrBech32Address(in.Config.Authority)
    }

    k := keeper.NewKeeper(in.Cdc, in.AddressCodec, in.StoreService, authority.String())
    m := NewAppModule(in.Cdc, k)

    return ModuleOutputs{Module: m, Keeper: k}
}
```

Fortunately it is possible to copy it.

</CodeGroupItem>
<CodeGroupItem title="autocli.go">

```go [https://github.com/b9lab/checkers-minimal/blob/module-files/module/autocli.go]
package module

import (
    autocliv1 "cosmossdk.io/api/cosmos/autocli/v1"
)

// AutoCLIOptions implements the autocli.HasAutoCLIConfig interface.
func (am AppModule) AutoCLIOptions() *autocliv1.ModuleOptions {
    return &autocliv1.ModuleOptions{
        Query: nil,
        Tx:    nil,
    }
}
```

At the moment there are no commands to define, so it stays simple.

</CodeGroupItem>
</CodeGroup>

---

Once more, you need to run:

```sh
$ go mod tidy
```

Your checkers module is not prepared as a standalone module. It is time to integrate it into `chain-minimal`.

## Integrate checkers in `chain-minimal`

You now have to direct `chain-minimal` to integrate your checkers module. Your checkers module is known with the `github.com/alice/checkers` package, and located in a parallel folder.

Direct chain-minimal to the right dependency, so it does not look for the module on `github.com`. Go to your `go.mod` and add:

```diff [https://github.com/b9lab/chain-minimal/blob/with-module/go.mod#L8]
...
    replace (
+      github.com/alice/checkers => ../checkers-minimal/
        // Fix upstream GHSA-h395-qcrw-5vmq vulnerability.
...
```

Now, you can define the module in the `app/app.yaml` file in two locations:

<CodeGroup>
<CodeGroupItem title="Name and type">

```diff-yaml [https://github.com/b9lab/chain-minimal/blob/with-module/app/app.yaml#L48-L50]
    ...
    - name: tx
      config:
        "@type": cosmos.tx.config.v1.Config
+  - name: checkers
+    config:
+      "@type": alice.checkers.module.v1.Module
    ...
```

Where you recall the Protobuf `alice.checkers.module.v1.Module` defined earlier:

</CodeGroupItem>
<CodeGroupItem title="Genesis">

```diff-yaml [https://github.com/b9lab/chain-minimal/blob/with-module/app/app.yaml#L13]
    ...
-  init_genesis: [auth, bank, distribution, staking, genutil]
+  init_genesis: [auth, bank, distribution, staking, genutil, checkers]
    ...
```

This is done because its genesis needs to be called too.

</CodeGroupItem>
</CodeGroup>

---

What remains is to update `app/app.go`, where you place your checkers keeper at the right locations:

```diff-go [https://github.com/b9lab/chain-minimal/blob/with-module/app/app.go]
    import(
        ...
+      checkerskeeper "github.com/alice/checkers/keeper"
        "github.com/cosmos/cosmos-sdk/baseapp"
        ...
        _ "cosmossdk.io/api/cosmos/tx/config/v1"          // import for side-effects
+      _ "github.com/alice/checkers/module"              // import for side-effects
        _ "github.com/cosmos/cosmos-sdk/x/auth"           // import for side-effects
        ...
    )
    ...
    type MiniApp struct {
        ...
        ConsensusParamsKeeper consensuskeeper.Keeper
+      CheckersKeeper        checkerskeeper.Keeper
        ...
    }
    ...
    func NewMiniApp(
        ...
    ) (*MiniApp, error) {
        ...
        if err := depinject.Inject(
            ...
            &app.ConsensusParamsKeeper,
+          &app.CheckersKeeper,
        ); err != nil {
            return nil, err
        }
        ...
    }
    ...
```

And that's about it. `depinject` and the rest take care of initialization and runtime.

<HighlightBox type="note">

The _side-effects_ comments refer to the `init()` function of Go packages. If you omit the new empty `import` call, the corresponding `init` function will not be called. Because of that, your `alice.checkers.module.v1.Module` Protobuf objects will not be known, which can lead to an unpleasant **runtime** error such as `unable to resolve "alice.checkers.module.v1.Module"`.

The missing object is only detected at runtime because its value is in `app.yaml`.

</HighlightBox>

## Run your checkers chain

Just like you did in the [previous section](./1-preparation.md), you compile the minimal chain, re-initialize, and start it. You need to re-initialize because your genesis has changed:

```sh
$ go mod tidy
$ make install
$ make init
$ minid start
```

There you have it: your minimal chain with a checkers module is running. After stopping it with <kbd>CTRL-C</kbd>, confirm that the checkers module was correctly integrated by calling up:

<CodeGroup>
<CodeGroupItem title="Straight">

```sh
$ minid export
```

</CodeGroupItem>
<CodeGroupItem title="Clean">

```sh
$ minid export | tail -n 1 | jq
```

</CodeGroupItem>
</CodeGroup>

In there, you can find:

```diff-json
    {
        ...
        "app_state: {
            ...
+          "checkers": {
+              "params": {}
+          },
            ...
        }
    }
```

This is as expected.

## Up next

Now that you have integrated your empty checkers module, it is time to make it interesting.
