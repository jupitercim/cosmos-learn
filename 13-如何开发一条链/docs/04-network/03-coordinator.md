---
sidebar_position: 3
description: Ignite Network commands for coordinators.
---

# 协调员指南

协调员在Ignite链上组织并启动新的链。

---

## 发布链

链启动过程的第一步是协调员发布启动链的意图。
`publish`命令从项目git仓库发布启动链的意图。

```shell
ignite n chain publish https://github.com/ignite/example --no-check
```

**输出**

```
✔ Source code fetched
✔ Blockchain set up
✔ Chain's binary built
✔ Blockchain initialized
✔ Genesis initialized
✔ Network published
⋆ Launch ID: 3
```

`LaunchID`标识了Ignite区块链上已发布的区块链。


```sh
ignite n chain list
```
### 指定初始创世

在协调过程中，新的创世账户和创世验证者被添加到链的创世文件中。
默认情况下，这些账户被添加到由链二进制文件生成的默认创世文件中。

协调员可以使用`--genesis`标志为链启动指定自定义的初始创世文件。这个自定义的初始创世文件可以包含额外的默认创世账户和链模块的自定义参数。

必须为`--genesis-url`标志提供一个URL。这个URL可以直接指向一个JSON格式的创世文件或一个包含创世文件的压缩包。

```shell
ignite n chain publish https://github.com/ignite/example --genesis-url https://raw.githubusercontent.com/ignite/example/master/genesis/gen.json
```

## 批准验证者请求

在协调链启动时，验证者会发送请求。这些请求代表希望成为链的创世验证者的请求。

协调员可以列出这些请求：

``` sh
ignite n request list 83
```

> **注意:** 这里的“3”指定的是`LaunchID`。

**输出**

```
Id  Status      Type                    Content
1  APPROVED     Add Genesis Account     spn1daefnhnupn85e8vv0yc5epmnkcr5epkqncn2le, 100000000stake
2  APPROVED     Add Genesis Validator   e3d3ca59d8214206839985712282967aaeddfb01@84.118.211.157:26656, spn1daefnhnupn85e8vv0yc5epmnkcr5epkqncn2le, 95000000stake
3  PENDING      Add Genesis Account     spn1daefnhnupn85e8vv0yc5epmnkcr5epkqncn2le, 95000000stake
4  PENDING      Add Genesis Validator   b10f3857133907a14dca5541a14df9e8e3389875@84.118.211.157:26656, spn1daefnhnupn85e8vv0yc5epmnkcr5epkqncn2le, 95000000stake
```

协调员可以批准或拒绝这些请求。

批准请求：

```
ignite n request approve 3 3,4
```

> **注意:** 在选择请求列表时，可以使用两种语法：`1,2,3,4`和`1-3,4`。

**输出**

```
✔ Source code fetched
✔ Blockchain set up
✔ Requests format verified
✔ Blockchain initialized
✔ Genesis initialized
✔ Genesis built
✔ The network can be started
✔ Request(s) #3, #4 verified
✔ Request(s) #3, #4 approved
```

Ignite CLI会自动验证这些请求是否可以应用于创世文件，批准的请求不会生成无效的创世文件。

拒绝请求：

```
ignite n request reject 3 3,4
```

**输出**

```
✔ Request(s) #3, #4 rejected
```

---

## 启动链

当创世文件中批准了足够多的验证者且协调员认为链已准备好启动时，协调员可以启动链。

此操作将最终确定链的创世文件，这意味着无法再为该链批准新的请求。

此操作还设置链的启动时间（或创世时间），即区块链网络上线的时间。

```
ignite n chain launch 3
```

**输出**

```
✔ Chain 3 will be launched on 2022-10-01 09:00:00.000000 +0200 CEST
```

这个示例输出显示了网络上链的启动时间。

### 设置自定义启动时间

默认情况下，启动时间将设置为最早可能的日期。实际上，验证者应该有时间为网络启动准备他们的节点。如果验证者未能上线，他们可能会因验证者集合中的不活跃而被监禁。

协调员可以使用`--launch-time`标志指定自定义时间。

```
ignite n chain launch --launch-time 2022-01-01T00:00:00Z
```