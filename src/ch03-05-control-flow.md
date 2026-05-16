## 控制流（Control Flow）

根据条件是否为 `true` 来执行某些代码，以及在条件为 `true` 时重复执行某些代码，这些都是大多数编程语言的基本构建块（building blocks）。让你控制 Rust 代码执行流的最常见结构是 `if` 表达式（expressions）和循环（loops）。

### `if` 表达式（`if` Expressions）

`if` 表达式允许你根据条件来分支代码。你提供一个条件，然后声明："如果满足此条件，则运行此代码块。如果不满足条件，则不运行此代码块。"

在你的 _projects_ 目录下创建一个名为 _branches_ 的新项目来探索 `if` 表达式。在 _src/main.rs_ 文件中输入以下内容：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

所有 `if` 表达式都以关键字 `if` 开头，后跟一个条件。在这个例子中，条件检查变量 `number` 的值是否小于 5。如果条件为 `true`，我们将要执行的代码块紧跟在条件之后放在花括号内。与 `if` 表达式中的条件关联的代码块有时被称为 _分支（arms）_，就像我们在第 2 章的[“将猜测的数字与秘密数字进行比较”][comparing-the-guess-to-the-secret-number]<!-- ignore --> 一节中讨论的 `match` 表达式中的分支一样。

我们还可以选择包含一个 `else` 表达式，我们在这里也这样做了，以便在条件计算结果为 `false` 时给程序一个备选的代码块来执行。如果你没有提供 `else` 表达式且条件为 `false`，程序将直接跳过 `if` 代码块并继续执行下一段代码。

尝试运行这段代码；你应该会看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

让我们尝试将 `number` 的值改为使条件为 `false` 的值，看看会发生什么：

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

再次运行该程序，并查看输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

同样值得注意的是，这段代码中的条件 _必须_ 是 `bool` 类型。如果条件不是 `bool` 类型，我们会得到一个错误。例如，尝试运行以下代码：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

这次 `if` 条件计算出的值是 `3`，Rust 抛出了一个错误：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

该错误表明 Rust 期望一个 `bool` 类型但得到了一个整数。与 Ruby 和 JavaScript 等语言不同，Rust 不会自动尝试将非布尔类型转换为布尔类型。你必须明确地始终为 `if` 提供一个布尔值作为其条件。例如，如果我们希望 `if` 代码块仅在一个数字不等于 `0` 时运行，我们可以将 `if` 表达式改为以下形式：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

运行这段代码将打印 `number was something other than zero`。

#### 使用 `else if` 处理多个条件（Handling Multiple Conditions with `else if`）

你可以通过在 `else if` 表达式中结合 `if` 和 `else` 来使用多个条件。例如：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

这个程序有四个可能的执行路径。运行它之后，你应该会看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

当这个程序执行时，它会依次检查每个 `if` 表达式，并执行第一个条件为 `true` 的分支体。请注意，尽管 6 可以被 2 整除，但我们没有看到输出 `number is divisible by 2`，也没有看到来自 `else` 块的 `number is not divisible by 4, 3, or 2` 文本。这是因为 Rust 只执行第一个为 `true` 条件对应的代码块，一旦找到这样一个条件，它甚至不会检查其余的条件。

使用过多的 `else if` 表达式会使你的代码变得杂乱，因此如果你有多个条件，你可能想要重构你的代码。第 6 章将介绍一种强大的 Rust 分支结构 `match`，适用于这些情况。

#### 在 `let` 语句中使用 `if`（Using `if` in a `let` Statement）

因为 `if` 是一个表达式，我们可以在 `let` 语句的右侧使用它来将结果赋值给一个变量，如示例 3-2 所示。

<Listing number="3-2" file-name="src/main.rs" caption="将 `if` 表达式的结果赋值给一个变量">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` 变量将根据 `if` 表达式的结果绑定到一个值。运行这段代码看看会发生什么：

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

请记住，代码块的计算结果是其中的最后一个表达式，而数字本身也是表达式。在这种情况下，整个 `if` 表达式的值取决于哪个代码块被执行。这意味着 `if` 的每个分支可能产生的结果值必须是相同类型；在示例 3-2 中，`if` 分支和 `else` 分支的结果都是 `i32` 整数。如果类型不匹配，如下面的示例所示，我们会得到一个错误：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

当我们尝试编译这段代码时，会得到一个错误。`if` 和 `else` 分支的值类型不兼容，Rust 会精确地指出问题在程序中的位置：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` 块中的表达式计算为一个整数，而 `else` 块中的表达式计算为一个字符串。这行不通，因为变量必须只有一个类型，并且 Rust 需要在编译时明确知道 `number` 变量的类型。知道 `number` 的类型可以让编译器验证该类型在我们使用 `number` 的任何地方都是有效的。如果 `number` 的类型只在运行时才能确定，Rust 将无法做到这一点；如果编译器必须跟踪任何变量的多种假设类型，它将变得更加复杂，并且对代码的保证也会更少。

### 使用循环重复执行（Repetition with Loops）

多次执行同一段代码通常非常有用。为此，Rust 提供了几种 _循环（loops）_，它们会运行循环体（loop body）内的代码直到末尾，然后立即从头开始。为了试验循环，让我们创建一个名为 _loops_ 的新项目。

Rust 有三种循环：`loop`、`while` 和 `for`。让我们逐一尝试。

#### 使用 `loop` 重复执行代码（Repeating Code with `loop`）

`loop` 关键字告诉 Rust 反复执行一段代码，要么永远执行下去，直到你明确告诉它停止。

例如，将 _loops_ 目录中的 _src/main.rs_ 文件修改为如下内容：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

当我们运行这个程序时，我们会看到 `again!` 不断地被打印出来，直到我们手动停止程序。大多数终端支持键盘快捷键 <kbd>ctrl</kbd>-<kbd>C</kbd> 来中断一个陷入持续循环的程序。试试看：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

符号 `^C` 表示你按下 <kbd>ctrl</kbd>-<kbd>C</kbd> 的位置。

你可能会或可能不会在 `^C` 之后看到 `again!` 被打印出来，这取决于代码在收到中断信号时处于循环的哪个位置。

幸运的是，Rust 也提供了一种通过代码跳出循环的方法。你可以在循环中使用 `break` 关键字来告诉程序何时停止执行循环。回想一下，我们在第 2 章的[“猜对后退出”][quitting-after-a-correct-guess]<!-- ignore --> 一节的猜谜游戏中就是这样做的，当用户猜对数字赢得游戏时退出程序。

我们还在猜谜游戏中使用了 `continue`，它在循环中告诉程序跳过本次循环迭代中剩余的代码，并进入下一次迭代。

#### 从循环中返回值（Returning Values from Loops）

`loop` 的用途之一是重试你知道可能会失败的操作，例如检查线程是否已完成其任务。你可能还需要将该操作的结果从循环中传递给你代码的其余部分。为此，你可以在用来停止循环的 `break` 表达式之后添加你想要返回的值；该值将从循环中被返回，以便你可以使用它，如下所示：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

在循环之前，我们声明了一个名为 `counter` 的变量并将其初始化为 `0`。然后，我们声明了一个名为 `result` 的变量来保存从循环返回的值。在循环的每次迭代中，我们给 `counter` 变量加 `1`，然后检查 `counter` 是否等于 `10`。当它等于 `10` 时，我们使用 `break` 关键字并带上值 `counter * 2`。循环之后，我们使用分号结束将值赋给 `result` 的语句。最后，我们打印 `result` 中的值，在本例中为 `20`。

你也可以在循环内部使用 `return`。`break` 只退出当前循环，而 `return` 总是退出当前函数。

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### 使用循环标签消除歧义（Disambiguating with Loop Labels）

如果你在循环中嵌套了循环，`break` 和 `continue` 将应用于该点最内层的循环。你可以选择在循环上指定一个 _循环标签（loop label）_，然后你可以将它与 `break` 或 `continue` 一起使用，以指定这些关键字应用于带标签的循环，而不是最内层的循环。循环标签必须以单引号开头。下面是一个包含两个嵌套循环的例子：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

外层循环有标签 `'counting_up`，它将从 0 计数到 2。没有标签的内层循环从 10 向下计数到 9。第一个没有指定标签的 `break` 只会退出内层循环。`break 'counting_up;` 语句将退出外层循环。这段代码会打印：

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### 使用 `while` 简化条件循环（Streamlining Conditional Loops with `while`）

程序通常需要在循环内评估一个条件。当条件为 `true` 时，循环运行。当条件不再为 `true` 时，程序调用 `break`，停止循环。可以使用 `loop`、`if`、`else` 和 `break` 的组合来实现这种行为；如果你愿意，现在可以尝试在程序中这样做。然而，这种模式非常常见，以至于 Rust 为此内置了一个语言结构，称为 `while` 循环。在示例 3-3 中，我们使用 `while` 将程序循环三次，每次倒计时，然后在循环结束后打印一条消息并退出。

<Listing number="3-3" file-name="src/main.rs" caption="当条件为 `true` 时使用 `while` 循环运行代码">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

这种结构消除了如果你使用 `loop`、`if`、`else` 和 `break` 所需的大量嵌套，并且更加清晰。当条件为 `true` 时，代码运行；否则，它退出循环。

#### 使用 `for` 遍历集合（Looping Through a Collection with `for`）

你可以选择使用 `while` 结构来遍历集合（如数组）中的元素。例如，示例 3-4 中的循环打印数组 `a` 中的每个元素。

<Listing number="3-4" file-name="src/main.rs" caption="使用 `while` 循环遍历集合中的每个元素">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

这里，代码逐个遍历数组中的元素。它从索引 `0` 开始，然后循环直到到达数组中的最后一个索引（即当 `index < 5` 不再为 `true` 时）。运行这段代码将打印数组中的每个元素：

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

所有五个数组值都按预期出现在终端中。尽管 `index` 在某个时刻会达到值 `5`，但循环在尝试从数组中获取第六个值之前就停止了执行。

然而，这种方法容易出错；如果索引值或测试条件不正确，我们可能会导致程序恐慌（panic）。例如，如果你将数组 `a` 的定义改为有四个元素，但忘记将条件更新为 `while index < 4`，代码就会 panic。它也很慢，因为编译器会添加运行时代码来在每次循环迭代中检查索引是否在数组的边界内。

作为一种更简洁的替代方案，你可以使用 `for` 循环为集合中的每个项目执行一些代码。`for` 循环看起来像示例 3-5 中的代码。

<Listing number="3-5" file-name="src/main.rs" caption="使用 `for` 循环遍历集合中的每个元素">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

当我们运行这段代码时，我们会看到与示例 3-4 相同的输出。更重要的是，我们现在提高了代码的安全性，消除了因超出数组末尾或未达到足够位置而遗漏某些项目而导致错误的可能性。由 `for` 循环生成的机器码也可能更高效，因为不需要在每次迭代时将索引与数组长度进行比较。

使用 `for` 循环，如果你更改了数组中值的数量，你不需要像示例 3-4 中使用的方法那样记住更改任何其他代码。

`for` 循环的安全性和简洁性使其成为 Rust 中最常用的循环结构。即使在你想要运行某段代码一定次数的情况下，比如示例 3-3 中使用 `while` 循环的倒计时示例，大多数 Rustacean（Rust 开发者）也会使用 `for` 循环。实现这一点的方法是使用标准库提供的 `Range`（范围），它按顺序生成从一个数字开始到另一个数字之前结束的所有数字。

以下是使用 `for` 循环和我们尚未讨论的另一种方法 `rev`（用于反转范围）来实现倒计时的样子：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

这段代码更优雅一些，不是吗？

## 总结（Summary）

你做到了！这是相当大的一章：你学习了变量（variables）、标量（scalar）和复合数据类型（compound data types）、函数（functions）、注释（comments）、`if` 表达式（expressions）和循环（loops）！为了练习本章讨论的概念，尝试编写程序来完成以下任务：

- 在华氏度和摄氏度之间转换温度。
- 生成第 *n* 个斐波那契数（Fibonacci number）。
- 打印圣诞颂歌《The Twelve Days of Christmas》的歌词，利用歌曲中的重复部分。

当你准备好继续前进时，我们将讨论 Rust 中一个在其他编程语言中 _不_ 常见的概念：所有权（ownership）。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
