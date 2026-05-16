## 变量（Variables）与可变性（Mutability）

正如在[“使用变量存储值”][storing-values-with-variables]<!-- ignore -->一节中提到的，默认情况下，
变量（variables）是不可变的（immutable）。这是 Rust 引导你利用其提供的安全性和轻松并发（concurrency）优势来编写代码的众多方式之一。
不过，你仍然可以选择让变量变为可变的（mutable）。
让我们来探讨 Rust 为何以及如何鼓励你倾向于使用不可变性（immutability），以及为什么有时你可能想要选择不使用它。

当一个变量不可变时，一旦值被绑定（bound）到一个名字上，你就无法改变
该值。为了说明这一点，在你的 _projects_ 目录下使用 `cargo new variables`
生成一个名为 _variables_ 的新项目。

然后，在你的新 _variables_ 目录中，打开 _src/main.rs_ 并将其代码替换为
以下代码，这段代码暂时还无法编译：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

使用 `cargo run` 保存并运行该程序。你应该会收到一条关于
不可变性（immutability）错误的错误信息，如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

这个例子展示了编译器如何帮助你找到程序中的错误。
编译器错误可能会令人沮丧，但实际上它们只是意味着你的程序
还没有安全地完成你想要它做的事情；它们并 _不_ 意味着你不是
一个优秀的程序员！经验丰富的 Rustaceans 也会遇到编译器错误。

你收到的错误信息是 `` cannot assign twice to immutable variable `x` ``（不能对不可变变量 `x` 二次赋值），因为你试图给不可变的 `x` 变量赋予第二个值。

当我们试图更改一个被指定为不可变的值时，能够在编译时（compile-time）得到错误是很重要的，因为这种情况正是 bug（程序错误）的源头。如果代码的一部分假定某个值永远不会改变，而另一部分代码改变了该值，那么第一部分代码可能就无法按预期工作了。这类 bug 的原因在事后很难追踪，尤其是当第二段代码只是 _有时_ 改变该值时。Rust 编译器保证，当你声明一个值不会改变时，它就真的不会改变，因此你无需自己追踪它。你的代码因此
更易于推理（reason）。

但是可变性（mutability）可能非常有用，并且可以让代码编写起来更便捷。
虽然变量默认是不可变的，但你仍然可以通过在变量名前添加 `mut` 来使其可变，就像你在[第 2 章][storing-values-with-variables]<!-- ignore -->中所做的那样。添加 `mut` 还向未来的代码阅读者传达了意图，表明代码的其他部分将会改变这个变量的值。

例如，让我们将 _src/main.rs_ 改为以下内容：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

现在当我们运行这个程序时，会得到如下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

当使用了 `mut` 时，我们可以将绑定到 `x` 的值从 `5` 改为 `6`。
最终，是否使用可变性取决于你，以及你认为在特定情况下哪种方式最清晰。

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### 声明常量（Constants）

与不可变变量类似，_常量（constants）_ 也是绑定到名字上且不允许改变的值，但常量与变量之间有几个不同之处。

首先，不允许对常量使用 `mut`。常量不仅仅是默认不可变——它们始终不可变。声明常量使用 `const` 关键字而不是 `let` 关键字，并且 _必须_ 标注值的类型。我们将在下一节 [“数据类型”][data-types]<!-- ignore -->中介绍类型和类型注解（type annotations），所以现在不必担心细节。只需知道你必须始终标注类型。

常量可以在任何作用域（scope）中声明，包括全局作用域（global scope），这使得它们对于代码中许多部分都需要知道的值非常有用。

最后一个区别是，常量只能设置为常量表达式（constant expression），而不能设置为只能在运行时计算得出的值。

以下是一个常量声明的示例：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

该常量的名称为 `THREE_HOURS_IN_SECONDS`，其值被设置为 60（一分钟的秒数）乘以 60（一小时的分钟数）再乘以 3（我们想计算的程序中的小时数）的结果。Rust 的常量命名约定（naming convention）是全部使用大写字母，单词之间用下划线分隔。编译器能够在编译时（compile time）评估有限的一组操作，这让我们可以选择以一种更易于理解和验证的方式写出这个值，而不是将该常量直接设置为 10,800。有关声明常量时可以使用哪些操作的更多信息，请参阅 [Rust 参考手册中关于常量求值][const-eval]的部分。

常量在其声明的整个作用域内，在程序运行的整个过程都有效。这一特性使得常量对于应用程序域中多个部分可能需要知道的值非常有用，例如游戏中任何玩家允许获得的最高分数，或者光速。

将程序中使用的硬编码（hardcoded）值命名为常量，有助于向未来的代码维护者传达该值的含义。如果将来需要更新这个硬编码的值，它也只需修改代码中的一个地方即可。

### 隐藏（Shadowing）

正如你在[第 2 章][comparing-the-guess-to-the-secret-number]<!-- ignore -->的猜谜游戏教程中看到的，你可以声明一个与之前变量同名的新变量。Rustaceans 说第一个变量被第二个变量 _隐藏（shadowed）_ 了，这意味着当你使用该变量名时，编译器将看到的是第二个变量。实际上，第二个变量遮蔽（overshadows）了第一个变量，将变量名的所有使用都指向自身，直到它自己被隐藏或作用域结束。我们可以通过使用相同的变量名并重复使用 `let` 关键字来隐藏变量，如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

这个程序首先将 `x` 绑定到值 `5`。然后，通过重复 `let x =` 创建了一个新的变量 `x`，将原始值加上 `1`，使得 `x` 的值变为 `6`。接着，在用花括号创建的内部作用域（inner scope）中，第三个 `let` 语句也隐藏了 `x` 并创建了一个新变量，将之前的值乘以 `2`，使 `x` 的值为 `12`。当该作用域结束时，内部隐藏结束，`x` 恢复为 `6`。当我们运行这个程序时，它会输出以下内容：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

隐藏（shadowing）与将变量标记为 `mut` 不同，因为如果我们不小心尝试在不使用 `let` 关键字的情况下重新赋值给这个变量，就会得到一个编译时错误。通过使用 `let`，我们可以对一个值进行几次变换（transformations），但在这些变换完成后，变量仍然保持不可变。

`mut` 与隐藏之间的另一个区别是，由于我们在再次使用 `let` 关键字时实际上是创建了一个新变量，因此我们可以改变值的类型（type），同时复用相同的名称。例如，假设我们的程序要求用户通过输入空格字符来显示他们希望在文本之间有多少空格，然后我们希望将该输入存储为一个数字：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

第一个 `spaces` 变量是字符串（string）类型，第二个 `spaces` 变量是数字（number）类型。因此，隐藏使我们不必想出不同的名称，比如 `spaces_str` 和 `spaces_num`；相反，我们可以复用更简单的 `spaces` 名称。然而，如果我们尝试对此使用 `mut`，如下所示，就会得到一个编译时错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

错误信息表明我们不允许改变变量的类型：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

现在我们已经探讨了变量的工作方式，接下来让我们看看变量可以拥有的更多数据类型（data types）。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
