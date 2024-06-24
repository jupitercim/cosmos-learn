```markdown
---
ics: 1
title: ICS 规范标准
stage: 草案
category: meta
kind: meta
author: Christopher Goes <cwgoes@tendermint.com>
created: 2019-02-12
modified: 2019-08-25
---

## 什么是 ICS?

跨链标准（ICS）是一份设计文档，描述了预计对 Cosmos 生态系统有用的特定协议、标准或功能。
ICS 应列出标准的期望特性，解释设计原理，并提供简洁而全面的技术规范。主要的 ICS 作者负责推动提案通过标准化过程，征求社区的意见和支持，并与相关利益相关者沟通，以确保（社会）共识。

跨链标准化过程应是提出生态系统范围内的协议、变更和功能的主要工具，而 ICS 文档应在共识达成后持续存在，作为设计决策的记录和未来实施者的信息库。

跨链标准 *不* 应用于提出特定区块链（如 Cosmos Hub）的变更、指定实现细节（如特定语言的数据结构）、或者就现有 Cosmos 区块链的治理提案进行辩论（尽管 Cosmos 生态系统中的个别区块链可能利用其治理流程来批准或拒绝跨链标准）。

## 组件

一个 ICS 包括标题、简介、规范、历史记录和版权声明。所有顶级部分都是必需的。
参考文献应作为链接内嵌或在必要时作为表格放置在该部分的底部。

### 标题

ICS 标题包含与 ICS 相关的元数据。

#### 必填字段

`ics: #` - ICS 编号（按顺序分配）

`title` - ICS 标题（简短明了）

`stage` - 当前 ICS 阶段，参见 [PROCESS.md](../../meta/PROCESS.md) 获取可能的阶段列表。

有关 ICS 阶段接受阶段的描述，请参见 [README.md](../../README.md)。

`category` - ICS 类别，以下之一：

- `meta` - 关于 ICS 过程的标准。
- `IBC/TAO` - 关于跨区块链通信系统核心传输、认证和排序层协议的标准。
- `IBC/APP` - 关于跨区块链通信系统应用层协议的标准。

`kind` - ICS 种类，以下之一：

- `meta` - 关于 ICS 过程的标准。
- `interface` - 关于最小接口集合的标准，由托管实施跨链通信协议状态机的实现必须提供和满足的属性。
- `instantiation` - 关于具体实现细节的标准，解释了如何在伪代码或软件组件中实现该标准。

`author` - ICS 作者及联系信息（优先顺序：电子邮件、GitHub 句柄、Twitter 句柄、其他联系方法可能会得到回应）。
           第一作者是主要的 ICS “所有者”，负责推进标准化过程。
           后续作者的排序应按照贡献量排序。

`created` - 创建 ICS 的日期（`YYYY-MM-DD`）

`modified` - 最后修改 ICS 的日期（`YYYY-MM-DD`）

#### 可选字段

`requires` - 其他 ICS 标准，按编号引用，此标准所需或依赖的。

`required-by` - 其他 ICS 标准，按编号引用，需要或依赖此标准的。

`replaces` - 由此标准替代或取代的另一个 ICS 标准（如果适用）。

`replaced-by` - 替换或取代此标准的另一个 ICS 标准（如果适用）。

`version compatibility` - 与 ICS 标准兼容的实现版本列表。

### 简介

在标题之后，ICS 应包括一个简要的（约 200 字）简介，提供该规范的高级描述和基本原理。

### 规范

规范部分是 ICS 的主要组成部分，应包含协议文档、设计原理、必需参考资料和适当的技术细节。

#### 子组件

规范可以包括以下任何或所有子组件，适用于特定的 ICS。包括的子组件应按照此处指定的顺序列出。

- *动机* - 对所提出特性存在的理由，或对现有特性变更的提议的理由。
- *定义* - 在此 ICS 中使用的新术语或概念列表，或者理解此 ICS 所需的任何术语。
- *期望特性* - 指定的协议或特性的期望特性或特征列表，以及在违反这些特性时预期的影响或失败。
- *技术规范* - 所提议协议的所有技术细节，包括语法、语义、子协议、数据结构、算法和必要时的伪代码。
  技术规范应详细到足以使不相互了解的独立正确实现该规范的实施是兼容的。
- *向后兼容性* - 与先前特性或协议版本的兼容性（或其缺乏兼容性）讨论。
- *向前兼容性* - 与预期的未来特性或协议版本的兼容性（或其缺乏兼容性）讨论。
- *示例实现* - 具体的示例实现或描述预期的实现，作为实施者的主要参考。
- *其他实现* - 候选或已完成的实现列表（外部引用，不是内联）。

### 历史

ICS 应包括一个历史部分，列出任何启发性文档，并记录重要更改的纯文本日志。

请参见下文的历史部分示例 [below](#history-1)。

### 版权

ICS 应包括一个版权部分，通过 [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) 放弃版权。

## 格式

### 通用

ICS 规范必须使用 GitHub 风格的 Markdown 编写。

有关 GitHub 风格的 Markdown 速查表，请参见 [here](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)。
有关本地 Markdown 渲染器，请参见 [here](https://github.com/joeyespo/grip)。

### 语言

ICS 规范应使用简单英语编写，避免晦涩术语和不必要的行话。有关简单英语的优秀示例，请参见 [Simple English Wikipedia](https://simple.wikipedia.org/wiki/Main_Page)。

在规范中，“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY” 和 “OPTIONAL” 等关键词应按照 [RFC 2119](https://tools.ietf.org/html/rfc2119) 中描述的方式解释。

### 伪代码

规范中的伪代码应是与语言无关的简单命令式标准，并包含行号、变量、简单条件块、for 循环和必要

时用于解释更多功能的英语片段，例如调度超时。应避免使用 LaTeX 图像，因为它们在差异形式中难以审查。

结构的伪代码应以简单的 Typescript 编写，作为接口。

示例结构的伪代码：

```typescript
interface Connection {
  state: ConnectionState
  version: Version
  counterpartyIdentifier: Identifier
  consensusState: ConsensusState
}
```

算法的伪代码应以简单的 Typescript 编写，作为函数。

示例算法的伪代码：

```typescript
function startRound(round) {
  round_p = round
  step_p = PROPOSE
  if (proposer(h_p, round_p) === p) {
    if (validValue_p !== nil)
      proposal = validValue_p
    else
      proposal = getValue()
    broadcast( {PROPOSAL, h_p, round_p, proposal, validRound} )
  } else
    schedule(onTimeoutPropose(h_p, round_p), timeoutPropose(round_p))
}
```

## 历史

本规范受以太坊的 [EIP 1](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1.md) 的显著启发和衍生，后者又源于比特币的 BIP 过程和 Python 的 PEP 过程。前人作者对本 ICS 规范或 ICS 过程的任何缺陷不负责。请将所有评论指向 ICS 存储库维护者。

2019 年 3 月 4 日 - 初稿完成并提交为 PR

2019 年 3 月 7 日 - 合并初稿

2019 年 4 月 11 日 - 更新伪代码格式，添加定义子部分

2019 年 8 月 17 日 - 对类别进行澄清

## 版权

本文档所有内容均在 [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) 下许可。
```

这是对 ICS 规范标准文档的翻译，包括标头、简介、规范、历史和版权部分的详细描述。