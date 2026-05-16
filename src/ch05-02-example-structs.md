## 一个使用结构体（Struct）的示例程序

为了理解何时使用结构体（Struct），让我们编写一个计算矩形面积（Area）的程序。我们将从使用单个变量开始，然后重构程序，直到最终使用结构体。

让我们用 Cargo 创建一个名为 _rectangles_ 的新二进制项目，它将接收以像素为单位指定的矩形的宽度（Width）和高度（Height），并计算矩形的面积。示例 5-8 展示了我们项目中 *src/main.rs* 的一个简短程序，它正是这样做的。

<Listing number="5-8" file-name="src/main.rs" caption="计算由单独的宽度和高度变量指定的矩形面积">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

现在，使用 `cargo run` 运行这个程序：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

这段代码通过使用每个维度调用 `area` 函数成功计算了矩形的面积，但我们还可以做更多来使这段代码清晰易读。

这个问题在 `area` 的签名中很明显：

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` 函数本应计算一个矩形的面积，但我们编写的函数有两个参数，而且在程序的任何地方都没有明确说明这些参数是相关的。将宽度和高度组合在一起会更易读和更易管理。我们在第 3 章的["元组类型"][the-tuple-type]<!-- ignore -->部分已经讨论过一种实现方式：使用元组（Tuple）。

### 使用元组（Tuple）重构

示例 5-9 展示了使用元组（Tuple）的另一个版本。

<Listing number="5-9" file-name="src/main.rs" caption="使用元组指定矩形的宽度和高度">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

从某方面来说，这个程序更好。元组（Tuple）让我们增加了一些结构，现在我们只传递一个参数。但从另一方面来说，这个版本不够清晰：元组（Tuple）没有命名其元素，所以我们不得不通过索引来访问元组的各个部分，这使得我们的计算不够直观。

混淆宽度和高度对面积计算来说无关紧要，但如果我们想在屏幕上绘制矩形，那就重要了！我们必须记住 `width` 是元组索引 `0`，`height` 是元组索引 `1`。如果其他人要使用我们的代码，这将更难让他们弄清楚并记住。因为我们没有在代码中传达数据的含义，所以现在更容易引入错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### 使用结构体（Struct）重构

我们使用结构体（Struct）通过标记数据来增加含义。我们可以将目前使用的元组（Tuple）转换为一个结构体，为整体以及各个部分都赋予名称，如示例 5-10 所示。

<Listing number="5-10" file-name="src/main.rs" caption="定义一个 `Rectangle` 结构体">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

这里，我们定义了一个名为 `Rectangle` 的结构体。在花括号内，我们将字段定义为 `width` 和 `height`，两者类型都是 `u32`。然后，在 `main` 中，我们创建了一个特定的 `Rectangle` 实例，其宽度为 `30`，高度为 `50`。

我们的 `area` 函数现在定义了一个参数，我们将其命名为 `rectangle`，其类型是 `Rectangle` 结构体实例的不可变借用。正如第 4 章中提到的，我们想要借用结构体而不是获取它的所有权。这样，`main` 保留了其所有权，可以继续使用 `rect1`，这就是我们在函数签名和调用函数时使用 `&` 的原因。

`area` 函数访问 `Rectangle` 实例的 `width` 和 `height` 字段（注意，访问借用的结构体实例的字段不会移动字段的值，这就是为什么你经常看到借用结构体的原因）。我们的 `area` 函数签名现在准确地表达了我们的意图：使用 `Rectangle` 的 `width` 和 `height` 字段计算其面积。这传达了宽度和高度是相互关联的，并且为值提供了描述性名称，而不是使用 `0` 和 `1` 这样的元组索引值。这在清晰性上是一个胜利。

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### 使用派生 trait（Derived Trait）增加功能

能够在调试程序时打印 `Rectangle` 的实例并查看其所有字段的值将非常有用。示例 5-11 尝试使用我们之前章节中用过的 [`println!` 宏][println]<!-- ignore -->。然而，这不会工作。

<Listing number="5-11" file-name="src/main.rs" caption="尝试打印一个 `Rectangle` 实例">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

当我们编译这段代码时，会得到以下核心错误信息：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` 宏可以做多种类型的格式化，默认情况下，花括号告诉 `println!` 使用一种称为 `Display` 的格式化方式：即直接供最终用户使用的输出。到目前为止我们看到的基本类型默认实现了 `Display`，因为向用户展示 `1` 或任何其他基本类型只有一种方式。但对于结构体（Struct），`println!` 应该以何种方式格式化输出就不那么明确了，因为存在更多的显示可能性：你想要逗号还是不想要？你想要打印花括号吗？应该显示所有字段吗？由于这种歧义，Rust 不会试图猜测我们想要什么，结构体也没有提供 `Display` 的实现供 `println!` 和 `{}` 占位符使用。

如果继续阅读错误信息，我们会发现这个有用的提示：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

让我们试试！`println!` 宏的调用现在看起来像 `println!("rect1 is {rect1:?}");`。将说明符 `:?` 放在花括号内告诉 `println!` 我们想要使用一种称为 `Debug` 的输出格式。`Debug` trait 使我们能够以对开发者有用的方式打印结构体，这样我们就可以在调试代码时看到它的值。

编译修改后的代码。哎呀！我们仍然得到一个错误：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

但同样，编译器给了我们一个有用的提示：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _确实_ 包含了打印调试信息的功能，但我们需要显式地选择加入，才能使该功能对我们的结构体可用。为此，我们在结构体定义之前添加外部属性 `#[derive(Debug)]`，如示例 5-12 所示。

<Listing number="5-12" file-name="src/main.rs" caption="添加属性以派生 `Debug` trait 并使用调试格式打印 `Rectangle` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

现在当我们运行程序时，不会得到任何错误，我们将看到以下输出：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

太好了！虽然它不是最漂亮的输出，但它显示了这个实例的所有字段的值，这在调试时肯定会有帮助。当我们有更大的结构体时，拥有更易读的输出会很有用；在这些情况下，我们可以在 `println!` 字符串中使用 `{:#?}` 而不是 `{:?}`。在这个例子中，使用 `{:#?}` 风格将输出以下内容：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

另一种使用 `Debug` 格式打印值的方法是使用 [`dbg!` 宏][dbg]<!-- ignore -->，它获取表达式的所有权（与 `println!` 相反，后者获取引用），打印代码中 `dbg!` 宏调用所在位置的文件名和行号以及该表达式的结果值，并返回该值的所有权。

> 注意：调用 `dbg!` 宏会打印到标准错误控制台流（`stderr`），而 `println!` 打印到标准输出控制台流（`stdout`）。我们将在第 12 章的["将错误信息重定向到标准错误"][err]<!-- ignore -->部分更多地讨论 `stderr` 和 `stdout`。

下面是一个示例，我们关心分配给 `width` 字段的值，以及 `rect1` 中整个结构体的值：

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

我们可以将 `dbg!` 放在表达式 `30 * scale` 周围，由于 `dbg!` 返回表达式的值的所有权，`width` 字段将获得与没有 `dbg!` 调用时相同的值。我们不希望 `dbg!` 获取 `rect1` 的所有权，所以我们在下一个调用中使用对 `rect1` 的引用。以下示例的输出看起来像这样：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

我们可以看到，第一行输出来自 _src/main.rs_ 的第 10 行，我们在那里调试表达式 `30 * scale`，其结果值为 `60`（为整数实现的 `Debug` 格式化只打印它们的值）。_src/main.rs_ 第 14 行的 `dbg!` 调用输出了 `&rect1` 的值，即 `Rectangle` 结构体。这个输出使用了 `Rectangle` 类型的漂亮 `Debug` 格式化。当你试图弄清楚代码在做什么时，`dbg!` 宏真的很有用！

除了 `Debug` trait 之外，Rust 还提供了许多 trait 供我们与 `derive` 属性一起使用，这些 trait 可以为我们的自定义类型添加有用的行为。这些 trait 及其行为列在[附录 C][app-c]<!-- ignore -->中。我们将在第 10 章中介绍如何以自定义行为实现这些 trait 以及如何创建自己的 trait。除了 `derive` 之外，还有许多其他属性；更多信息，请参阅 [Rust 参考手册的"属性"部分][attributes]。

我们的 `area` 函数非常特定：它只计算矩形的面积。将这个行为更紧密地绑定到我们的 `Rectangle` 结构体上会很有帮助，因为它不适用于任何其他类型。让我们看看如何通过将 `area` 函数转换为在 `Rectangle` 类型上定义的 `area` 方法来继续重构这段代码。

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
