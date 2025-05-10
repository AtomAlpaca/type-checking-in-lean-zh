# 什么是 **内核**？

内核是一段在软件中实现的 Lean 逻辑——也就是一段**最小化机制**的计算机程序，能够构造 Lean 逻辑语言中的对象，并验证这些对象的正确性。它的主要组成部分包括：

* **名字（name）的种类**，用于寻址。
* **宇宙层级（universe level）的种类**。
* **表达式（expression）的种类**（λ-抽象、变量等）。
* **声明（declaration）的种类**（公理、定义、定理、归纳类型等）。
* **环境（environment）**：名字到声明的映射。
* **表达式操作功能**，例如束缚变量替换、宇宙参数替换。
* **类型检查核心操作**：类型推断、化简、判定定义等价。
* **归纳类型相关的操作与检验**：生成类型的递归器（消除规则），以及检查构造子的定义是否与类型规范一致。
* **可选的内核扩展**，使上述操作能够直接处理 `nat` 与 `string` 字面量。

之所以要隔离一个小型内核，并要求 Lean 中的定义被翻译成一种**最简内核语言**，目的在于提高证明系统的可信度。Lean 的设计允许使用者在功能完备的证明助理中工作，享受强大的元编程、丰富的编辑器支持和可扩展语法；同时又能把构造出的证明项导出为一种形式，使人们**无需信任那些实现高级特性的代码**，也能独立验证其正确性——这些高级特性正是 Lean（作为证明助理）高效且易用的原因。

在 [*Certified Programming with Dependent Types*](http://adam.chlipala.net/cpdt/) 一书第 1.2.3 节中，Adam Chlipala 提出了通常称为 \*\*de Bruijn 准则（de Bruijn criterion）\*\*或 **de Bruijn 原则**的概念：

> 当证明助理即便采用复杂且可扩展的策略来搜索证明，也会输出可在小型内核语言中表达的证明项时，就满足了 “de Bruijn 准则”。这些核心语言的特性复杂度大致与形式化数学基础（如 ZF 集合论）的提案相当。要相信一个证明，我们可以无视搜索过程中的潜在错误，只依赖一个（相对较小的）**证明检查内核**来验证搜索结果。

Lean 的内核足够小，小到开发者可以自行实现，并借助 **导出器（exporter）**[^1] 独立检查 Lean 中的证明。Lean 的导出格式为每个声明提供了充分信息，使得用户可以选择只实现完整内核的一部分。例如，若只对类型推断、化简和定义等价的核心功能感兴趣，就可以省略归纳类型规范检查的实现。

除了上面列出的项目，外部类型检查器还需要一个针对 [Lean 导出格式](./export_format.md) 的**解析器**以及一个**美化打印器**，分别用于输入和输出。解析器与打印器并不属于内核，但若想与内核进行有意义的交互，它们就不可或缺。

[^1]: 编写自己的类型检查器并非“下午茶”级别的小项目，但对于热爱探索的技术爱好者而言，绝对在可实现的范围内。


# What is the kernel?

The kernel is an implementation of Lean's logic in software; a computer program with the minimum amount of machinery required to construct elements of Lean's logical language and check those elements for correctness. The major components are:

+ A sort of names used for addressing.

+ A sort of universe levels.

+ A sort of expressions (lambdas, variables, etc.)

+ A sort of declarations (axioms, definitions, theorems, inductive types, etc.)

+ Environments, which are maps of names to declarations.

+ Functionality for manipulating expressions. For example bound variable substitution and substitution of universe parameters.

+ Core operations used in type checking, including type inference, reduction, and definitional equality checking.

+ Functionality for manipulating and checking inductive type declarations. For example, generating a type's recursors (elimination rules), and checking whether a type's constructors agree with the type's specification.

+ Optional kernel extensions which permit the operations above to be performed on nat and string literals.

The purpose of isolating a small kernel and requiring Lean definitions to be translated to a minimal kernel language is to increase the trustworthiness of the proof system. Lean's design allows users to interact with a full-featured proof assistant which offers nice things like robust metaprogramming, rich editor support, and extensible syntax, while also permitting extraction of constructed proof terms into a form that can be verified without having to trust the correctness of the code that implements the higher level features that makes Lean (the proof assistant) productive and pleasant to use.

In section 1.2.3 of the [_Certified Programming with Dependent Types_](http://adam.chlipala.net/cpdt/), Adam Chlipala defines what is sometimes referred to as the de Bruijn criterion, or de Bruijn principle.

> Proof assistants satisfy the “de Bruijn criterion” when they produce proof terms in small kernel languages, even when they use complicated and extensible procedures to seek out proofs in the first place. These core languages have feature complexity on par with what you find in proposals for formal foundations for mathematics (e.g., ZF set theory). To believe a proof, we can ignore the possibility of bugs during search and just rely on a (relatively small) proof-checking kernel that we apply to the result of the search.

Lean's kernel is small enough that developers can write their own implementation and independently check proofs in Lean by using an exporter[^1]. Lean's export format contains enough information about the exported declarations that users can optionally restrict their implementation to certain subsets of the full kernel. For example, users interested in the core functionality of inference, reduction, and definitional equality may opt out of implementing the functionality for checking inductive specifications.

In addition to the list of items above, external type checkers will also need a parser for [Lean's export format](./export_format.md), and a pretty printer, for input and output respectively. The parser and pretty printer are not part of the kernel, but they are important if one wants to have interesting interactions with the kernel.

[^1]: Writing your own type checker is not an afternoon project, but it is well within the realm of what is achievable for citizen scientists.
