## 数据类型（Data Types）

Rust 中的每个值都属于某种特定的**数据类型**（_data type_），这告诉 Rust 它所处理的是哪种数据，从而知道该如何处理这些数据。我们将讨论两种数据类型子集：标量类型（scalar）和复合类型（compound）。

请记住，Rust 是一门**静态类型**（_statically typed_）语言，这意味着它在编译时必须知道所有变量的类型。编译器通常可以根据值及其使用方式推断出我们想使用的类型。在可能存在多种类型的情况下，例如我们在第 2 章的[“将猜测的数字与秘密数字进行比较”][comparing-the-guess-to-the-secret-number]<!-- ignore -->章节中，使用 `parse` 将 `String` 转换为数值类型时，我们必须添加一个类型注解（type annotation），如下所示：

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

如果我们在前面的代码中没有添加 `: u32` 类型注解，Rust 将显示以下错误，这意味着编译器需要更多信息来确定我们想要使用哪种类型：

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

对于其他数据类型，你也会看到不同的类型注解。

### 标量类型（Scalar Types）

**标量**（_scalar_）类型表示单个值。Rust 有四种主要的标量类型：整数（integer）、浮点数（floating-point number）、布尔值（Boolean）和字符（character）。你可能从其他编程语言中认识它们。让我们来看看它们在 Rust 中是如何工作的。

#### 整数类型（Integer Types）

**整数**（_integer_）是一个没有小数部分的数字。我们在第 2 章中使用了一种整数类型 `u32`。这种类型声明表明与其关联的值应该是一个占用 32 位空间的无符号整数（有符号整数类型以 `i` 开头，而不是 `u`）。表 3-1 展示了 Rust 中内置的整数类型。我们可以使用其中任何一种变体来声明整数值的类型。

<span class="caption">表 3-1：Rust 中的整数类型</span>

| 长度   | 有符号（Signed） | 无符号（Unsigned） |
| ------- | ------- | -------- |
| 8 位   | `i8`    | `u8`     |
| 16 位  | `i16`   | `u16`    |
| 32 位  | `i32`   | `u32`    |
| 64 位  | `i64`   | `u64`    |
| 128 位 | `i128`  | `u128`   |
| 依架构而定（Architecture-dependent） | `isize` | `usize`  |

每种变体可以是有符号（signed）或无符号（unsigned），并且具有明确的大小。**有符号**（_signed_）和**无符号**（_unsigned_）指的是数字是否可能为负数——换句话说，就是数字是否需要带符号（有符号），还是它只会是正数因此可以不带符号表示（无符号）。这就像在纸上写数字：当符号很重要时，数字会带有加号或减号；然而，当可以安全地假定数字为正数时，就会不带符号显示。有符号数以[补码][twos-complement]<!-- ignore -->形式存储。

每个有符号变体可以存储从 −(2<sup>n − 1</sup>) 到 2<sup>n − 1</sup> − 1（含）的数字，其中 _n_ 是该变体使用的位数。因此，`i8` 可以存储从 −(2<sup>7</sup>) 到 2<sup>7</sup> − 1 的数字，即 −128 到 127。无符号变体可以存储从 0 到 2<sup>n</sup> − 1 的数字，因此 `u8` 可以存储从 0 到 2<sup>8</sup> − 1 的数字，即 0 到 255。

此外，`isize` 和 `usize` 类型取决于程序所运行的计算机架构：在 64 位架构上是 64 位，在 32 位架构上是 32 位。

你可以用表 3-2 中所示的任何一种形式来书写整数字面量（integer literal）。请注意，可以属于多种数值类型的数字字面量允许使用类型后缀（type suffix），例如 `57u8`，来指定类型。数字字面量还可以使用 `_` 作为视觉分隔符，使数字更易于阅读，例如 `1_000`，它与指定 `1000` 具有相同的值。

<span class="caption">表 3-2：Rust 中的整数字面量</span>

| 数字字面量（Number literals） | 示例（Example）       |
| ---------------- | ------------- |
| 十进制（Decimal）          | `98_222`      |
| 十六进制（Hex）              | `0xff`        |
| 八进制（Octal）            | `0o77`        |
| 二进制（Binary）           | `0b1111_0000` |
| 字节（Byte，仅 `u8`） | `b'A'`        |

那么如何知道该使用哪种整数类型呢？如果你不确定，Rust 的默认值通常是不错的起点：整数类型默认为 `i32`。使用 `isize` 或 `usize` 的主要情况是在对某种集合进行索引时。

> ##### 整数溢出（Integer Overflow）
>
> 假设你有一个类型为 `u8` 的变量，它可以保存 0 到 255 之间的值。如果你试图将该变量更改为超出该范围的值（例如 256），就会发生**整数溢出**（_integer overflow_），这可能会导致两种行为之一。当你在调试模式（debug mode）下编译时，Rust 会包含整数溢出检查，如果发生这种情况，会导致程序在运行时**恐慌**（_panic_）。当程序因错误而退出时，Rust 使用术语 _panicking_（恐慌）；我们将在第 9 章的[“使用 `panic!` 的不可恢复错误”][unrecoverable-errors-with-panic]<!-- ignore -->章节中更深入地讨论恐慌。
>
> 当你使用 `--release` 标志在发布模式（release mode）下编译时，Rust **不会**包含导致恐慌的整数溢出检查。相反，如果发生溢出，Rust 会执行**补码环绕**（_two's complement wrapping_）。简而言之，大于该类型能容纳的最大值的值会“绕回”到该类型能容纳的最小值。以 `u8` 为例，值 256 变为 0，值 257 变为 1，依此类推。程序不会恐慌，但变量的值很可能不是你期望的值。依赖整数溢出的环绕行为被认为是一个错误。
>
> 为了显式地处理溢出的可能性，你可以使用标准库为原始数值类型提供的以下几类方法：
>
> - 在所有模式下使用 `wrapping_*` 方法进行环绕，例如 `wrapping_add`。
> - 如果发生溢出则返回 `None` 值，使用 `checked_*` 方法。
> - 返回值和一个指示是否发生溢出的布尔值（Boolean），使用 `overflowing_*` 方法。
> - 在值的最小或最大值处饱和（saturate），使用 `saturating_*` 方法。

#### 浮点类型（Floating-Point Types）

Rust 还有两种**浮点数**（_floating-point numbers_）的原始类型，即带有小数点的数字。Rust 的浮点类型是 `f32` 和 `f64`，分别占 32 位和 64 位。默认类型是 `f64`，因为在现代 CPU 上，它的速度与 `f32` 大致相同，但精度更高。所有浮点类型都是有符号的。

下面是一个展示浮点数用法的示例：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

浮点数按照 IEEE-754 标准表示。

#### 数值运算（Numeric Operations）

Rust 支持你对所有数字类型所期望的基本数学运算：加法（addition）、减法（subtraction）、乘法（multiplication）、除法（division）和取余（remainder）。整数除法会向零截断（truncates toward zero）到最接近的整数。以下代码展示了如何在 `let` 语句中使用每种数值运算：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

这些语句中的每个表达式都使用一个数学运算符，并求值为单个值，然后绑定到一个变量。[附录 B][appendix_b]<!-- ignore -->包含了 Rust 提供的所有运算符的列表。

#### 布尔类型（The Boolean Type）

与大多数其他编程语言一样，Rust 中的布尔类型有两个可能的值：`true` 和 `false`。布尔值的大小为一个字节。Rust 中的布尔类型使用 `bool` 指定。例如：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

使用布尔值的主要方式是通过条件表达式，例如 `if` 表达式。我们将在[“控制流”][control-flow]<!-- ignore -->章节中介绍 `if` 表达式在 Rust 中是如何工作的。

#### 字符类型（The Character Type）

Rust 的 `char` 类型是该语言最原始的字母类型。以下是一些声明 `char` 值的示例：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

需要注意的是，我们使用单引号指定 `char` 字面量，而字符串字面量则使用双引号。Rust 的 `char` 类型大小为 4 字节，表示一个 Unicode 标量值（Unicode scalar value），这意味着它可以表示比 ASCII 多得多的内容。重音字母；中文、日文和韩文字符；表情符号（emoji）；以及零宽空格（zero-width space）在 Rust 中都是有效的 `char` 值。Unicode 标量值的范围是从 `U+0000` 到 `U+D7FF` 以及从 `U+E000` 到 `U+10FFFF`（含）。然而，“字符”在 Unicode 中并不是一个真正的概念，所以你对“字符”的人类直觉可能与 Rust 中的 `char` 并不完全一致。我们将在第 8 章的[“使用字符串存储 UTF-8 编码的文本”][strings]<!-- ignore -->中详细讨论这个主题。

### 复合类型（Compound Types）

**复合类型**（_Compound types_）可以将多个值组合成一个类型。Rust 有两种原始的复合类型：元组（tuple）和数组（array）。

#### 元组类型（The Tuple Type）

**元组**（_tuple_）是一种将多个不同类型的值组合成一个复合类型的通用方式。元组具有固定长度：一旦声明，它们不能增长或缩小。

我们通过编写一个括号内的逗号分隔的值列表来创建元组。元组中的每个位置都有一个类型，并且元组中不同值的类型不必相同。在这个示例中，我们添加了可选的类型注解：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

变量 `tup` 绑定到整个元组，因为元组被认为是一个单一的复合元素。要从元组中取出各个值，我们可以使用模式匹配（pattern matching）来解构（destructure）元组值，如下所示：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

这个程序首先创建一个元组并将其绑定到变量 `tup`。然后它使用带有 `let` 的模式来获取 `tup` 并将其转换为三个独立的变量 `x`、`y` 和 `z`。这被称为**解构**（_destructuring_），因为它将单个元组拆分为三个部分。最后，程序打印 `y` 的值，即 `6.4`。

我们还可以直接使用句点（`.`）后跟要访问的值的索引来访问元组元素。例如：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

这个程序创建了元组 `x`，然后使用各自的索引访问元组的每个元素。与大多数编程语言一样，元组中的第一个索引是 0。

没有任何值的元组有一个特殊的名称，即**单元**（_unit_）。这个值及其对应的类型都写作 `()`，表示一个空值或空返回类型。如果表达式不返回任何其他值，则隐式返回单元值。

#### 数组类型（The Array Type）

另一种拥有多个值的集合方式是使用**数组**（_array_）。与元组不同，数组中的每个元素必须具有相同的类型。与其他一些编程语言中的数组不同，Rust 中的数组具有固定长度。

我们以方括号内的逗号分隔列表的形式来编写数组的值：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

当你希望数据分配在栈（stack）上而不是堆（heap）上时（我们将在[第 4 章][stack-and-heap]<!-- ignore -->中更详细地讨论栈和堆），或者当你希望确保始终有固定数量的元素时，数组非常有用。不过，数组不像 vector（向量）类型那样灵活。vector 是标准库提供的一种类似的集合类型，它的大小是允许增长或缩小的，因为其内容位于堆上。如果你不确定应该使用数组还是 vector，那么很可能应该使用 vector。[第 8 章][vectors]<!-- ignore -->会更详细地讨论 vector。

然而，当你确定元素的数量不需要改变时，数组更有用。例如，如果你在程序中使用月份的名称，你可能更倾向于使用数组而不是 vector，因为你清楚它总是包含 12 个元素：

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

你可以使用方括号来编写数组的类型，其中包含每个元素的类型、分号以及数组中的元素数量，如下所示：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

这里，`i32` 是每个元素的类型。分号后面的数字 `5` 表示数组包含五个元素。

你还可以通过指定初始值，后跟分号，然后在方括号中指定数组的长度，来将数组初始化为每个元素都包含相同的值，如下所示：

```rust
let a = [3; 5];
```

名为 `a` 的数组将包含 `5` 个元素，这些元素最初都将被设置为值 `3`。这与编写 `let a = [3, 3, 3, 3, 3];` 相同，但写法更简洁。

<!-- 旧标题。请勿删除，否则链接可能会失效。 -->
<a id="accessing-array-elements"></a>

#### 访问数组元素（Array Element Access）

数组是一块已知固定大小的连续内存，可以在栈上分配。你可以使用索引来访问数组中的元素，如下所示：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

在这个示例中，名为 `first` 的变量将获得值 `1`，因为这是数组中索引 `[0]` 处的值。名为 `second` 的变量将从数组中的索引 `[1]` 获得值 `2`。

#### 无效的数组元素访问（Invalid Array Element Access）

让我们看看如果尝试访问超出数组末尾的元素会发生什么。假设你运行这段代码（类似于第 2 章中的猜数游戏），从用户那里获取一个数组索引：

<span class="filename">文件名（Filename）: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

这段代码可以成功编译。如果你使用 `cargo run` 运行这段代码并输入 `0`、`1`、`2`、`3` 或 `4`，程序会打印出数组中该索引处的相应值。如果你输入一个超出数组末尾的数字，例如 `10`，你将看到类似如下的输出：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

程序在使用无效值进行索引操作时发生了运行时错误。程序退出并显示一条错误消息，且没有执行最后的 `println!` 语句。当你尝试使用索引访问元素时，Rust 会检查你指定的索引是否小于数组长度。如果索引大于或等于长度，Rust 将发生恐慌（panic）。这个检查必须在运行时进行，尤其是在这种情况下，因为编译器无法知道用户稍后运行代码时会输入什么值。

这是 Rust 内存安全原则的实际体现。在许多底层语言中，不会进行这种检查，当你提供不正确的索引时，可能会访问到无效的内存。Rust 通过立即退出而不是允许内存访问并继续执行，来保护你免受此类错误的影响。第 9 章将进一步讨论 Rust 的错误处理，以及如何编写既不会恐慌也不会允许无效内存访问的可读、安全的代码。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
