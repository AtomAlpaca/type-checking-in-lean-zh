# 非安全（unsafe）声明

Lean 的表层语言允许用户编写带有 `unsafe` 标记的声明，这类声明能执行通常被禁止的操作。例如，Lean 接受下面的定义：

```lean
unsafe def y : Nat := y
```

非安全声明不会被导出 [^note1]，因此也无需被信任；并且，即便在表层语言里，它们也不能出现在正式证明中。之所以仍允许写 `unsafe` 声明，是为了让用户在编写**生成证明的辅助代码**（本身不一定是证明）时拥有更大的自由度。

`aesop` 库提供了一个现实范例。[Aesop](https://github.com/leanprover-community/aesop) 是一个自动化框架，用来帮助用户生成证明。开发过程中，作者发现用**互递归归纳类型**表达系统的某部分最合适，[代码见此](https://github.com/leanprover-community/aesop/blob/69404390bdc1de946bf0a2e51b1a69f308e56d7a/Aesop/Tree/Data.lean#L375)。但这一组归纳类型在 Lean 理论里存在不合法的自引用，不会被内核接受，因此必须标记为 `unsafe`。

允许将该定义作 `unsafe` 声明是一种双赢：Aesop 开发者得以继续在 Lean 中用熟悉的语法实现库，而无需另学一套元编程 DSL，也不必为取悦内核大费周章；而 Aesop 的使用者仍能导出并验证 **Aesop 生成的** 证明，而无需验证 Aesop 自身。

[^note1]: 从技术上说，无法绝对阻止把 `unsafe` 声明写进导出文件（导出器本身并非可信组件），但内核在加载时会检查这些声明，若它们确实不安全，就不会把它们加入环境。若类型检查器收到含上述 Aesop 代码的导出文件，应当报错并拒绝加载。


# Unsafe declarations

Lean's vernacular allows users to write declarations marked as `unsafe`, which are permitted to do things that are normally forbidden. For example, Lean permits the following definition:

```
  unsafe def y : Nat := y
```

Unsafe declarations are not exported[^note1], do not need to be trusted, and (for the record) are not permitted in proofs, even in the vernacular. Permitting unsafe declarations in the vernacular is still beneficial for Lean users, because it gives users more freedom when writing code that is used to produce proofs but doesn't have to be a proof in and of itself.

The aesop library provides us with an excellent real world example. [Aesop](https://github.com/leanprover-community/aesop) is an automation framework; it helps users generate proofs. At some point in development, the authors of aesop felt that the best way to express a certain part of their system was with a mutually defined inductive type, [seen here](https://github.com/leanprover-community/aesop/blob/69404390bdc1de946bf0a2e51b1a69f308e56d7a/Aesop/Tree/Data.lean#L375). It just so happens that this set of inductive type has an invalid occurrence of one of the types being declared within Lean's theory, and would not be permitted by Lean's kernel, so it needs to be marked `unsafe`.

Permitting this definition as an `unsafe` declaration is still a win-win. The Aesop developers were able to use Lean to write their library the way they wanted, in Lean, without having to call out to (and learn) a separate metaprogramming DSL, they didn't have to jump through hoops to satisfy the kernel, and users of aesop can still export and verify the proofs produced *by* aesop without having to verify aesop itself.

[^note1]: There's technically nothing preventing an unsafe declaration from being put in an export file (especially since the exporter is not a trusted component), but checks run by the kernel will prevent unsafe declarations from being added to the environment if they are actually unsafe. A properly implemented type checker would throw an error if it received an export file declaring the aesop library code described above.
