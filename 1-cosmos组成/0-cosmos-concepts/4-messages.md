---
title: "Messages"
order: 5
description: Introduction to MsgService and the flow of messages
tags:
  - concepts
  - cosmos-sdk
---

# Messages

<HighlightBox type="prerequisite">

It is recommended to take a look at the following previous sections to better understand messages:

* [A Blockchain App Architecture](1-architecture.md)
* [Accounts](2-accounts.md)
* [Transactions](3-transactions.md)

</HighlightBox>

<HighlightBox type="learning">

In this section, you will take a closer look at messages, `Msg`. At the end of the section, you can find a code example that illustrates message creation and the inclusion of messages in transactions for your checkers blockchain.
<br/><br/>
Understanding `Msg` will help you prepare for the next section, on [modules in the Cosmos SDK](5-modules.md), as messages are a primary object handled by modules.

</HighlightBox>

Messages are one of two primary objects handled by a module in the Cosmos SDK. The other primary object handled by modules is queries. While messages inform the state and have the potential to alter it, queries inspect the module state and are always read-only.

In the Cosmos SDK, a **transaction** contains **one or more messages**. Modules process the messages after the transaction is included in a block by the consensus layer.

<ExpansionPanel title="Signing a message">

Remember from the [previous section on transactions](3-transactions.md) that transactions must be signed before a validator includes them in a block. Every message in a transaction must be signed by the addresses as specified by `GetSigners`.
<br/><br/>
The Cosmos SDK currently allows signing transactions with either `SIGN_MODE_DIRECT` or `SIGN_MODE_LEGACY_AMINO_JSON` methods.
<br/><br/>
When an account signs a message it signs an array of bytes. This array of bytes is the outcome of serializing the message. For the signature to be verifiable at a later date, this conversion needs to be deterministic. For this reason, you define a canonical bytes-representation of the message, typically with the parameters ordered alphabetically.

</ExpansionPanel>

## Messages and the transaction lifecycle

Transactions containing one or more valid messages are serialized and confirmed by CometBFT. As you might recall, CometBFT is agnostic to the transaction interpretation and has absolute finality. When a transaction is included in a block, it is confirmed and finalized with no possibility of chain re-organization or cancellation.

The confirmed transaction is relayed to the Cosmos SDK application for interpretation. Each message is routed to the appropriate module via `BaseApp` using `MsgServiceRouter`. `BaseApp` decodes each message contained in the transaction. Each module has its own `MsgService` that processes each received message.

## `MsgService`

Although it is technically feasible to proceed to create a novel `MsgService`, the recommended approach is to define a Protobuf `Msg` service. Each module has exactly one Protobuf `Msg` service defined in `tx.proto` and there is an RPC service method for each message type in the module. The Protobuf message service implicitly defines the interface layer of the state, mutating processes contained within the module.

How does all of this translate into code? Here is an example `MsgService` from the [`bank` module](https://docs.cosmos.network/v0.46/modules/bank/):

```protobuf
// Msg defines the bank Msg service.
service Msg {
  // Send defines a method for sending coins from one account to another account.
  rpc Send(MsgSend) returns (MsgSendResponse);

  // MultiSend defines a method for sending coins from some accounts to other accounts.
  rpc MultiSend(MsgMultiSend) returns (MsgMultiSendResponse);
}
```

In this example:

* Each `Msg` service method has exactly **one argument**, such as `MsgSend`, which must implement the `sdk.Msg` interface and a Protobuf response.
* The **standard naming convention** is to call the RPC argument `Msg<service-rpc-name>` and the RPC response `Msg<service-rpc-name>Response`.

## Client and server code generation

The Cosmos SDK uses Protobuf definitions to generate client and server code:

* The `MsgServer` interface defines the server API for the `Msg` service. Its implementation is described in the [`Msg` services documentation](https://docs.cosmos.network/v0.45/building-modules/msg-services.html).
* Structures are generated for all RPC requests and response types.

<HighlightBox type="docs">

If you want to dive deeper when it comes to messages, the `Msg` service, and modules, see:

* The Cosmos SDK documentation on [`Msg` service](https://docs.cosmos.network/v0.45/building-modules/msg-services.html).
* The Cosmos SDK documentation on messages and queries, addressing how to define messages using `Msg` services - [Amino `LegacyMsg`](https://docs.cosmos.network/v0.45/building-modules/messages-and-queries.html#legacy-amino-legacymsgs).

</HighlightBox>

## CLI

For the command-line interface (CLI), module developers create subcommands to add as children to the module-level message commands. These commands describe how to craft a message for inclusion in a transaction.

<HighlightBox type="tip">

With v0.50, the Cosmos SDK introduces the [`autocli` facility](https://docs.cosmos.network/v0.50/learn/advanced/autocli#module-wiring--customization). This takes care of a lot of the boilerplate and lets you define the available CLI commands in a descriptive manner.

</HighlightBox>

## Code example

<ExpansionPanel title="Show me some code for my checkers blockchain - Including messages">

In the [previous](3-transactions.md) design exercise's code examples, the ABCI application was aware of a single transaction type: that of a checkers move with four `int` values. With multiple games, this is no longer sufficient. Additionally, you need to conform to the SDK's way of handling `Tx`, which means **creating messages that are then included in a transaction**.

If you want the guided coding exercise instead of design and implementation considerations, see the links at the bottom of the page.
<br/><br/>
**What you need**

Begin by describing the messages you need for your checkers application to have a solid starting point before diving into the code:

1. In the former _Play_ transaction, your four integers need to move from the transaction to an `sdk.Msg`, wrapped in said transaction. Four flat `int` values are no longer sufficient, as you need to follow the `sdk.Msg` interface, identify the game for which a move is meant, and distinguish a move message from other message types.
2. You need to add a message type for creating a new game. When this is done, a player can create a new game which mentions other players. A generated (possibly) ID identifies this newly created game and is returned to the message creator.
3. It would be a good feature for the other person to be able to reject the challenge. This would have the added benefit of clearing the state of stale, unstarted games.

**How to proceed**

Focus on the messages around the **game creation**. There is no single true way of deciding what goes into your messages. The following is one reasonable example.

1. The message itself is structured like this:

    ```go
    type MsgCreateGame struct {
        Creator string
        Black   string
        Red     string
    }
    ```

    Note that `Creator` contains the address of the message signer.

2. The corresponding response message would then be:

    ```go
    type MsgCreateGameResponse struct {
        GameIndex string
    }
    ```

    The idea here is that when the creator does not know the ID of the game that will be created, the ID needs to be returned.

With the messages defined, you need to declare how the message should be handled. This involves:

1. Describing how the messages are deterministically serialized. For this, you use [Protobuf](/academy/2-cosmos-concepts/6-protobuf.md).
2. Writing the code that handles the message and places the new game in the storage.
3. Putting hooks and callbacks at the right places in the general message handling.

**How to proceed with v0.50 or after**

The SDK takes care of a lot of things. In particular, by annotating your Protobuf message, you can have it implement `sdk.Msg` after compilation. For instance:

```protobuf
message MsgCreateGame {
  option (cosmos.msg.v1.signer) = "creator";

  string creator = 1;
  string black = 2;
  string red = 3;
}
```

Then, you declare a Protobuf message server:

```protobuf
service Msg {
  option (cosmos.msg.v1.service) = true;

  rpc CreateGame(MsgCreateGame)
    returns (MsgCreateGameResponse);
}
```

After compilation, you register this together with your interface registry so that the module knows to send any `MsgCreateGame` to the `CreateGame` function of the `MsgClient` interface.

You also define a local `msgServer` that implements the `CreateGame` function with the required actions, such as:

* Extracting and verifying the players and other values from the message.
* Creating a new game and saving it to storage.
* Returning the expected message.

**How to proceed with v0.47 or earlier and Ignite**

Thinking from design to implementation, Ignite CLI can help you create these elements, plus the `MsgCreateGame` and `MsgCreateGameResponse` objects, with this command:

```sh
$ ignite scaffold message createGame black red \
    --module checkers \
    --response gameIndex
```

<HighlightBox type="info">

Ignite CLI creates a variety of other files. See [Run Your Own Cosmos Chain](/hands-on-exercise/1-ignite-cli/index.md) for details, and to make additions to existing files.

</HighlightBox>

_**A sample of things Ignite CLI does for you**_

Ignite CLI significantly reduces the amount of work a developer has to do to build an application with the Cosmos SDK. Among others, it:

* Assists with making sure that your message conforms to `sdk.Msg` by adding the boilerplate for `GetSigners` and `GetSignBytes`.
* Adjusts the module's code so that this new message type is handled when received, and adds an empty function where you introduce the action this message is supposed to trigger. It even places a helpful `// TODO: Handling the message` where you should place your code.

Ignite CLI is opinionated in terms of which files it creates to separate which concerns. If you are not using it, you are free to create the files you want.

_**What is left to do?**_

Your work is mostly done. You want to create the specific game creation code to replace `// TODO: Handling the message`. For this, you need to:

1. Decide how to create a new and unique game ID: `newIndex`.
    <br/><br/>

    <HighlightBox type="info">

    For more details, and to avoid diving too deep in this section, see:

    * [Run Your Own Cosmos Chain](/hands-on-exercise/1-ignite-cli/index.md) to start the guided coding exercise from scratch,
    * [Create and Save a Game Properly](/hands-on-exercise/1-ignite-cli/5-create-handling.md) to jump straight where you handle the new message.

    </HighlightBox>

2. Extract and verify addresses from the message.
3. Create a game object with all required parameters and save it to storage
4. And finally, return the expected message.

<HighlightBox type="remember">

Remember, as a part of good design practice:

* If you encounter an internal error (one that denotes an error in logic or catastrophic failure), you should `panic("This situation should not happen")`.
* If you encounter a user or _regular_ error (like a user not having enough funds), you should return a regular `error`.

</HighlightBox>

**Other considerations**

What would happen if one of the two players has accepted the game by playing, but the other player has neither accepted nor rejected the game? You can address this scenario by:

* Having a timeout after which the game is canceled.
* Keeping an index as a First-In-First-Out (FIFO) list, or a list of unstarted games ordered by their cancellation time, so that this automatic trigger does not consume too many resources.

What would happen if a player stops taking turns? To ensure functionality for your checkers application, you can consider:

* Having a timeout after which the game is forfeited. You could also automatically charge the forgetful player, if and when you implement a wager system. For the guided coding exercise on this part, head straight to [Keep an Up-To-Date Game Deadline](/hands-on-exercise/2-ignite-cli-adv/1-game-deadline.md).
* Keeping an index of games that could be forfeited. If both timeouts are the same, you can keep a single FIFO list of games so you can clear them from the top of the list as necessary. For the guided coding exercise on this part, head straight to [Put Your Games in Order](/hands-on-exercise/2-ignite-cli-adv/3-game-fifo.md).
* Handling the cancelation in ABCI's `EndBlock` (or rather its equivalent in the Cosmos SDK) without any of the players having to trigger the cancelation. For the guided coding exercise on this part, head straight to [Auto-Expiring Games](/hands-on-exercise/2-ignite-cli-adv/4-game-forfeit.md).

In general terms, you could add `timeout: Timestamp` to your `StoredGame` and update it every time something changes in the game. You can decide on a maximum delay, for example _one day_.
<br/><br/>

<HighlightBox type="info">

There are no _open_ challenges, meaning a player cannot create a game where the second player is unknown until someone steps in. Therefore, player matching is left outside of the blockchain. The enterprising student can incorporate it inside the blockchain by changing the necessary models.

</HighlightBox>

</ExpansionPanel>

<HighlightBox type="tip">

If you would like to get started on building your own checkers game, you can go straight to the main exercise in Run Your Own Cosmos Chain, either [natively with SDK v0.50](/hands-on-exercise/0-native/index.md) or [with Ignite CLI](/hands-on-exercise/1-ignite-cli/index.md) to start from scratch.

More specifically, you can jump to:

* [Create one Custom Message](/hands-on-exercise/0-native/4-add-message.md) to see how to simply create and handle a `MsgCreateGame` message natively with SDK v0.50.
* [Create Custom Messages](/hands-on-exercise/1-ignite-cli/4-create-message.md) to see how to simply create the `MsgCreateGame` with Ignite CLI.
* [Create and Save a Game Properly](/hands-on-exercise/1-ignite-cli/5-create-handling.md) to see how to handle `MsgCreateGame` created with Ignite CLI.
* [Add a Way to Make a Move](/hands-on-exercise/1-ignite-cli/6-play-game.md) for the same but with `MsgPlayMove`, still with Ignite CLI.

</HighlightBox>

<HighlightBox type="synopsis">

To summarize, this section has explored:

* Messages, one of two primary objects handled by a module in the Cosmos SDK, which inform the state and have the potential to alter it.
* How one or more messages form a transaction in the Cosmos SDK, and messages are only processed after a transaction is signed by a validator and included in a block by the consensus layer.
* An example of more complex message handling capabilities related to the checkers game blockchain.

</HighlightBox>

<!--## Next up

Look at the above code example to get a better sense of how theory translates into development. If you feel ready to dive into the next main concept of the Cosmos SDK, you can go directly to the [next section](./5-modules.md) to learn more about modules.-->
