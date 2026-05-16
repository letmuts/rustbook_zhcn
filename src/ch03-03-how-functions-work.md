## 函数（Functions）

函数（Functions）在 Rust 代码中随处可见。你已经见过这门语言中最重要的函数之一：`main` 函数，它是许多程序的入口点（entry point）。你也见过 `fn` 关键字，它用来声明新函数。

Rust 代码使用 _蛇形命名法（snake case）_ 作为函数和变量名称的常规风格，即所有字母都是小写，单词之间用下划线分隔。下面是一个包含示例函数定义的程序：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

在 Rust 中，我们通过输入 `fn` 后跟函数名和一对圆括号来定义函数。花括号告诉编译器函数体（function body）的开始和结束位置。

我们可以通过输入函数名后跟一对圆括号来调用任何已定义的函数。因为 `another_function` 在程序中被定义了，所以它可以从 `main` 函数内部被调用。注意，我们在源代码中是在 `main` 函数 _之后_ 定义 `another_function` 的；我们也可以将其定义在之前。Rust 不关心你在哪里定义函数，只要它们定义在调用者能看到的某个作用域（scope）中即可。

让我们新建一个名为 _functions_ 的二进制项目（binary project）来进一步探索函数。将 `another_function` 示例放入 _src/main.rs_ 中并运行它。你应该会看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

各行代码按照它们在 `main` 函数中出现的顺序执行。首先打印 "Hello, world!" 消息，然后调用 `another_function` 并打印其消息。

### 参数（Parameters）

我们可以定义带有 _参数（parameters）_ 的函数，参数是函数签名（signature）的一部分的特殊变量。当一个函数有参数时，你可以为这些参数提供具体的值。从技术上讲，这些具体的值被称为 _实参（arguments）_，但在日常交流中，人们往往会混用 _parameter_ 和 _argument_ 这两个词来指代函数定义中的变量或调用函数时传入的具体值。

在这个版本的 `another_function` 中，我们添加了一个参数：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

尝试运行这个程序；你应该会得到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function` 的声明有一个名为 `x` 的参数。`x` 的类型被指定为 `i32`。当我们向 `another_function` 传入 `5` 时，`println!` 宏会将 `5` 放入格式字符串中含有一对花括号 `x` 的位置。

在函数签名（function signatures）中，你 _必须_ 声明每个参数的类型。这是 Rust 设计中的一个深思熟虑的决定：要求在函数定义中进行类型注解（type annotation），意味着编译器几乎不需要你在代码的其他地方使用它们来推断你的类型。如果编译器知道函数期望什么类型，它也能给出更有帮助的错误信息。

当定义多个参数时，使用逗号分隔参数声明，就像这样：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

这个例子创建了一个名为 `print_labeled_measurement` 的函数，它有两个参数。第一个参数名为 `value`，类型是 `i32`。第二个参数名为 `unit_label`，类型是 `char`。然后该函数打印包含 `value` 和 `unit_label` 的文本。

让我们尝试运行这段代码。将当前 _functions_ 项目的 _src/main.rs_ 文件中的程序替换为前面的示例，然后使用 `cargo run` 运行它：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

因为我们用 `5` 作为 `value` 的值，用 `'h'` 作为 `unit_label` 的值调用了该函数，所以程序输出包含了这些值。

### 语句与表达式（Statements and Expressions）

函数体（Function bodies）由一系列语句（statements）组成，并可选择以一个表达式（expression）结尾。到目前为止，我们介绍的函数都没有包含结尾表达式，但你已经见过作为语句一部分的表达式了。因为 Rust 是一门基于表达式（expression-based）的语言，这是一个需要理解的重要区别。其他语言没有同样的区分，所以让我们来看看什么是语句和表达式，以及它们的差异如何影响函数体。

- _语句（Statements）_ 是执行某些操作但不返回值的指令。
- _表达式（Expressions）_ 会计算并产生一个结果值。

让我们看一些例子。

实际上，我们已经使用过语句和表达式了。使用 `let` 关键字创建变量并为其赋值是一个语句。在示例 3-1 中，`let y = 6;` 就是一个语句。

<Listing number="3-1" file-name="src/main.rs" caption="包含一个语句的 `main` 函数声明">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

函数定义也是语句；整个前面的例子本身就是一个语句。（不过，正如我们稍后将看到的，调用函数并不是一个语句。）

语句不返回值。因此，你不能将一个 `let` 语句赋值给另一个变量，就像下面的代码尝试做的那样；你会得到一个错误：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

当你运行这个程序时，你会看到如下错误：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` 语句不返回值，所以 `x` 没有可以绑定的东西。这与 C 和 Ruby 等其他语言不同，在这些语言中，赋值会返回赋值的值。在这些语言中，你可以写 `x = y = 6` 并让 `x` 和 `y` 都有值 `6`；但在 Rust 中并非如此。

表达式会计算出一个值，并且构成了你在 Rust 中编写的大部分其余代码。考虑一个数学运算，比如 `5 + 6`，这是一个计算结果为 `11` 的表达式。表达式可以是语句的一部分：在示例 3-1 中，语句 `let y = 6;` 中的 `6` 就是一个计算结果为 `6` 的表达式。调用函数是一个表达式。调用宏也是一个表达式。用花括号创建的新作用域块（scope block）也是一个表达式，例如：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

这个表达式：

```rust,ignore
{
    let x = 3;
    x + 1
}
```

是一个代码块，在这个例子中，它的计算结果为 `4`。该值作为 `let` 语句的一部分被绑定到 `y`。注意 `x + 1` 这一行末尾没有分号，这与你目前看到的大多数行不同。表达式不包含结尾的分号。如果你在表达式末尾加上分号，你就把它变成了一个语句，那么它将不再返回值。在接下来探索函数返回值（return values）和表达式时，请记住这一点。

### 具有返回值的函数（Functions with Return Values）

函数可以向调用它的代码返回值。我们不给返回值命名，但必须在箭头（`->`）之后声明其类型。在 Rust 中，函数的返回值（return value）与函数体（body）块中最后一个表达式的值同义。你可以通过使用 `return` 关键字并指定一个值来提前从函数返回，但大多数函数会隐式地返回最后一个表达式。下面是一个返回值的函数示例：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

在 `five` 函数中没有函数调用、宏、甚至 `let` 语句——只有数字 `5` 本身。这在 Rust 中是一个完全有效的函数。注意，函数的返回类型也被指定为 `-> i32`。尝试运行这段代码；输出应该如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five` 中的 `5` 就是函数的返回值，这就是为什么返回类型是 `i32`。让我们更详细地分析一下。这里有两点很重要：首先，`let x = five();` 这一行表明我们正在使用函数的返回值来初始化一个变量。因为函数 `five` 返回 `5`，所以这一行等同于以下代码：

```rust
let x = 5;
```

其次，`five` 函数没有参数并定义了返回值的类型，但函数体只有一个孤零零的 `5`，没有分号，因为它是一个表达式，我们想要返回它的值。

让我们看另一个例子：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

运行这段代码将打印 `The value of x is: 6`。但是，如果我们在包含 `x + 1` 的那一行末尾加上一个分号，把它从一个表达式（expression）改为一个语句（statement），会发生什么呢？

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

编译这段代码会产生一个错误，如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

主要的错误信息 `mismatched types`（类型不匹配）揭示了这段代码的核心问题。函数 `plus_one` 的定义说明它将返回一个 `i32`，但语句不会计算出一个值，而是表达为 `()`，即单元类型（unit type）。因此，什么也没有返回，这与函数定义相矛盾并导致了一个错误。在这段输出中，Rust 提供了一条可能有助于修正此问题的消息：它建议删除分号，这将修复这个错误。
