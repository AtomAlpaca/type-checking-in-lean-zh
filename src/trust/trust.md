# 信任

Lean 的核心价值之一在于它能够构建数学证明，包括关于程序正确性的证明。用户经常提出的一个问题是：信任 Lean 究竟需要多大程度的信任，以及具体需要信任哪些部分。

这个问题的答案包含两个方面：用户需要信任哪些部分才能相信Lean中的证明，以及用户需要信任哪些部分才能相信通过编译Lean程序获得的可执行程序。

具体来说，区别在于：证明（包括关于程序的陈述）和未编译的程序可以直接用Lean的内核语言表达，并由内核对实现进行检查。它们不需要被编译成可执行文件，因此信任仅限于检查它们的内核实现，而Lean编译器不属于可信代码库的一部分。

信任已编译Lean程序的正确性需要信任Lean的编译器，而编译器与内核是分离的，不属于Lean的核心逻辑。信任Lean中_关于程序的陈述_与信任_Lean编译器生成的程序_是两回事。关于Lean程序的陈述是证明，属于仅需信任内核的范畴。而信任关于程序的证明_能推广到已编译程序的行为_则会将编译器纳入可信代码库。

**注意**：策略（tactics）和其他元程序（metaprograms），即使是已编译的策略，也_完全不需要_被信任；它们是非可信代码，仅用于生成供其他部分使用的内核项。命题`P`可以通过任意复杂的已编译元程序在Lean中证明，而无需将可信代码库扩展到内核之外，因为元程序必须生成用Lean内核语言表达的证明。

+ 这些陈述适用于[导出](../export_format.md)的证明。为了让更~~挑剔~~谨慎的读者满意，这确实需要在某种程度上信任其他部分，例如运行导出器和验证器的计算机操作系统、硬件等。

+ 对于未导出的证明，用户还需要额外信任内核之外的Lean组件（如 elaborator、解析器等）。

# Trust

A big part of Lean's value proposition is the ability to construct mathematical proofs, including proofs about program correctness. A common question from users is how much trust, and in what exactly, is involved in trusting Lean.

An answer to this question has two parts: what users need to trust in order to trust proofs in Lean, and what users need to trust in order to trust executable programs obtained by compiling a Lean program. 

Concretely, the distinction is that proofs (which includes statements about programs) and uncompiled programs can be expressed directly in Lean's kernel language and checked by an implementation of the kernel. They do not need to be compiled to an executable, therefore the trust is limited to whatever implementation of the kernel they're being checked with, and the Lean compiler does not become part of the trusted code base.

Trusting the correctness of compiled Lean programs requires trust in Lean's compiler, which is separate from the kernel and is not part of Lean's core logic. There is a distinction between trusting _statements about programs_ in Lean, and trusting _programs produced by the Lean compiler_. Statements about Lean programs are proofs, and fall into the category that only requires trust in the kernel. Trusting that proofs about a program _extend to the behavior of a compiled program_ brings the compiler into the trusted code base.

**NOTE**: Tactics and other metaprograms, even tactics that are compiled, do *not* need to be trusted _at all_; they are untrusted code which is used to produce kernel terms for use by something else. A proposition `P` can be proved in Lean using an arbitrarily complex compiled metaprogram without expanding the trusted code base beyond the kernel, because the metaprogram is required to produce a proof expressed in Lean's kernel language.

+ These statements hold for proofs that are [exported](../export_format.md). To satisfy more ~~pedantic~~ vigilant readers, this does necessarily entail some degree of trust in, for example, the operating system on the computer used to run the exporter and verifier, the hardware, etc.

+ For proofs that are not exported, users are additionally trusting the elements of Lean outside the kernel (the elaborator, parser, etc.).

## A more itemized list

A more itemized description of the trust involved in Lean 4 comes from a post by Mario Carneiro on the Lean Zulip. 

> In general:
> 
> 1. You trust that the lean logic is sound (author's note: this would include any kernel extensions, like those for Nat and String)
> 
> 2. If you didn't prove the program correct, you trust that the elaborator has converted your input into the lean expression denoting the program you expect. 
> 
> 3. If you did prove the program correct, you trust that the proofs about the program have been checked (use external checkers to eliminate this)
> 
> 4. You trust that the hardware / firmware / OS software running all of these things didn't break or lie to you
> 
> 5. (When running the program) You trust that the hardware / firmware / OS software faithfully executes the program according to spec and there are no debuggers or magnets on the hard drive or cosmic rays messing with your output
>
> For compiled executables:
>
> 6. You trust that any compiler overrides (extern / implemented_by) do not violate the lean logic (i.e. the model matches the implementation)
>
> 7. You trust the lean compiler (which lowered the lean code to C) to preserve the semantics of the program
>
> 8. You trust clang / LLVM to convert the C program into an executable with the same semantics

The first set of points applies to both proofs and compiled executables, while the second set applies specifically to compiled executable programs.

## Trust for external checkers

1. You're still trusting Lean's logic is sound.

2. You're trusting that the developers of the external checker properly implemented the program.

3. You're trusting the implementing language's compiler or interpreter. If you run multiple external checkers, you can think of them as circles in a venn diagram; you're trusting that the part where the circles intersect is free of soundness issues.

4. For the Nat and String kernel extensions, you're probably trusting a bignum library and the UTF-8 string type of the implementing language.

The advantages of using external checkers are:

+ Users can check their results with something that is completely disjoint from the Lean ecosystem, and is not dependent on any parts of Lean's code base.

+ External checkers can be written to take advantage of mature compilers or interpreters.

+ For kernel extensions, users can cross-check the results of multiple bignum/string implementations.

+ Using the export feature is the only way to get out of trusting the parts of Lean outside the kernel, so there's a benefit to doing this even if the export file is checked by something like [lean4lean](https://github.com/digama0/lean4lean/tree/master). Users worried about fallout from misuse of Lean's metaprogramming features are therefore encouraged to use the export feature.

## 更详细的清单

关于Lean 4中信任问题的更详细说明来自Mario Carneiro在Lean Zulip上的帖子。

> 一般来说：
> 
> 1. 你需要信任Lean的逻辑是可靠的（作者注：这包括任何内核扩展，例如Nat和String的扩展）
> 
> 2. 如果你没有证明程序的正确性，你需要信任elaborator已将你的输入转换为符合预期的Lean表达式
> 
> 3. 如果你确实证明了程序的正确性，你需要信任关于程序的证明已被检查（可通过外部检查器消除此需求）
> 
> 4. 你需要信任运行这些程序的硬件/固件/操作系统软件没有出错或欺骗你
> 
> 5. （运行程序时）你需要信任硬件/固件/操作系统软件能按照规范忠实地执行程序，并且没有调试器、硬盘上的磁铁或宇宙射线干扰输出
>
> 对于已编译的可执行文件：
>
> 6. 你需要信任任何编译器覆盖（extern / implemented_by）没有违反Lean逻辑（即模型与实现匹配）
>
> 7. 你需要信任Lean编译器（将Lean代码降级为C代码）能保持程序的语义
>
> 8. 你需要信任clang/LLVM能将C程序转换为具有相同语义的可执行文件

第一组要点适用于证明和已编译的可执行文件，而第二组专门针对已编译的可执行程序。

## 对外部检查器的信任

1. 你仍然需要信任Lean的逻辑是可靠的。

2. 你需要信任外部检查器的开发者正确实现了该程序。

3. 你需要信任实现语言的编译器或解释器。如果运行多个外部检查器，你可以将它们视为维恩图中的圆圈；你需要信任这些圆圈重叠的部分没有可靠性问题。

4. 对于Nat和String的内核扩展，你可能需要信任一个大数库和实现语言的UTF-8字符串类型。

使用外部检查器的优势包括：

+ 用户可以用完全独立于Lean生态系统的工具检查结果，不依赖于Lean代码库的任何部分。

+ 外部检查器可以利用成熟的编译器或解释器实现。

+ 对于内核扩展，用户可以交叉检查多个大数/字符串实现的结果。

+ 使用导出功能是摆脱对Lean内核之外部分信任的唯一方法，因此即使导出文件是通过[lean4lean](https://github.com/digama0/lean4lean/tree/master)等工具检查的，这样做也有好处。因此，担心滥用Lean元编程功能可能带来影响的用户被鼓励使用导出功能。