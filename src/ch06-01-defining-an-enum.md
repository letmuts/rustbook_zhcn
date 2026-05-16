## 定义枚举（Defining an Enum）

结构体（Struct）为你提供了一种将相关字段和数据组合在一起的方式，比如 `Rectangle` 及其 `width` 和 `height`，
而枚举（Enum）则为你提供了一种表示某个值是一组可能取值中的一个的方式。
例如，我们可能想说 `Rectangle` 是一组可能的形状之一，其中还包括 `Circle` 和 `Triangle`。
为此，Rust 允许我们将这些可能性编码为一个枚举。

让我们看一个我们可能需要在代码中表达的场景，并了解为什么在这种情况下枚举比结构体更有用且更合适。
假设我们需要处理 IP 地址。目前，有两种主要的 IP 地址标准：第四版（Version Four）和第六版（Version Six）。
由于这些是我们的程序可能遇到的仅有的 IP 地址可能性，我们可以*枚举（Enumerate）*所有可能的变体（Variant），
这正是"枚举"这个名称的由来。

任何 IP 地址要么是第四版地址，要么是第六版地址，但不能同时是两者。
IP 地址的这一特性使得枚举这种数据结构非常合适，因为枚举值只能是其变体之一。
第四版和第六版地址从根本上说仍然是 IP 地址，因此当代码处理适用于任何一种 IP 地址的情况时，它们应该被视为同一种类型。

我们可以在代码中通过定义一个 `IpAddrKind` 枚举并列出 IP 地址可能的类型——`V4` 和 `V6`——来表达这个概念。
这些就是该枚举的变体（Variant）：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` 现在是一个自定义数据类型（Custom Data Type），我们可以在代码的其他地方使用它。

### 枚举值（Enum Values）

我们可以像下面这样创建 `IpAddrKind` 的两个变体的实例：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

请注意，枚举的变体被命名空间化（Namespaced）在其标识符之下，我们使用双冒号来分隔两者。
这很有用，因为现在 `IpAddrKind::V4` 和 `IpAddrKind::V6` 这两个值都属于同一类型：`IpAddrKind`。
然后，我们可以例如定义一个接受任何 `IpAddrKind` 的函数：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

我们可以使用任一变体调用此函数：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

使用枚举还有更多优势。进一步思考我们的 IP 地址类型，目前我们没有办法存储实际的 IP 地址*数据（Data）*；
我们只知道它的*类型（Kind）*是什么。鉴于你刚刚在第 5 章中学到了结构体，你可能会倾向于用结构体来解决这个问题，
如示例 6-1 所示。

<Listing number="6-1" caption="使用 `struct` 存储 IP 地址的数据和 `IpAddrKind` 变体">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

在这里，我们定义了一个结构体 `IpAddr`，它有两个字段：一个类型为 `IpAddrKind`（我们之前定义的枚举）的 `kind` 字段，
和一个类型为 `String` 的 `address` 字段。我们有两个该结构体的实例。
第一个是 `home`，它的 `kind` 值为 `IpAddrKind::V4`，关联的地址数据为 `127.0.0.1`。
第二个实例是 `loopback`。它的 `kind` 值为 `IpAddrKind` 的另一个变体 `V6`，并关联了地址 `::1`。
我们使用结构体将 `kind` 和 `address` 值捆绑在一起，因此现在变体与值相关联了。

然而，仅使用枚举来表示相同的概念更加简洁：与其在结构体中包含枚举，我们可以将数据直接放入每个枚举变体中。
这个新的 `IpAddr` 枚举定义表示 `V4` 和 `V6` 两个变体都将拥有关联的 `String` 值：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

我们将数据直接附加到枚举的每个变体上，因此不需要额外的结构体。
在这里，也更容易看到枚举工作方式的另一个细节：我们定义的每个枚举变体的名称同时也会变成一个函数，
用于构造该枚举的实例。也就是说，`IpAddr::V4()` 是一个函数调用，它接受一个 `String` 参数并返回一个 `IpAddr` 类型的实例。
作为定义枚举的结果，我们会自动获得这个构造函数。

使用枚举而不是结构体还有另一个优势：每个变体可以拥有不同类型和数量的关联数据。
第四版 IP 地址总是有四个数值分量，其值介于 0 到 255 之间。
如果我们想将 `V4` 地址存储为四个 `u8` 值，但仍将 `V6` 地址表示为一个 `String` 值，用结构体是做不到的。
枚举可以轻松处理这种情况：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

我们已经展示了几种不同的方式来定义数据结构来存储第四版和第六版 IP 地址。
然而，事实证明，存储 IP 地址并编码其类型是如此常见，以至于[标准库已经有一个我们可以使用的定义！][IpAddr]<!-- ignore -->
让我们看看标准库是如何定义 `IpAddr` 的。它具有与我们定义和使用的完全相同的枚举和变体，
但它将地址数据以两个不同结构体的形式嵌入到变体中，这两个结构体针对每个变体有不同的定义：

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

这段代码说明你可以在枚举变体中放入任何类型的数据：例如字符串、数值类型或结构体。
你甚至可以包含另一个枚举！此外，标准库中的类型通常不会比你想到的要复杂多少。

请注意，尽管标准库中包含 `IpAddr` 的定义，我们仍然可以创建和使用自己的定义而不会产生冲突，
因为我们尚未将标准库中的定义引入我们自己的作用域（Scope）。我们将在第 7 章中详细讨论将类型引入作用域。

让我们再看看示例 6-2 中的另一个枚举示例：这个枚举的变体中嵌入了多种多样的类型。

<Listing number="6-2" caption="一个 `Message` 枚举，其每个变体存储不同数量和类型的值">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

这个枚举有四个变体，具有不同的类型：

- `Quit`：完全没有关联数据
- `Move`：有命名字段（Named Field），就像结构体一样
- `Write`：包含一个单独的 `String`
- `ChangeColor`：包含三个 `i32` 值

定义像示例 6-2 中那样的带有变体的枚举，类似于定义不同类型的结构体定义，
只不过枚举不使用 `struct` 关键字，而且所有变体都组合在 `Message` 类型之下。
以下结构体可以持有与前面枚举变体相同的数据：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

但是如果我们使用这些不同的结构体，每个结构体都有自己的类型，那么我们就无法像使用示例 6-2 中定义的
`Message` 枚举（它是一个单一类型）那样轻松地定义一个函数来接收任何这些类型的消息。

枚举和结构体之间还有一个相似之处：正如我们可以使用 `impl` 在结构体上定义方法一样，
我们也可以在枚举上定义方法。下面是一个我们可以在 `Message` 枚举上定义的名为 `call` 的方法：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

方法体将使用 `self` 来获取我们调用该方法的值。在这个例子中，我们创建了一个变量 `m`，
其值为 `Message::Write(String::from("hello"))`，当 `m.call()` 运行时，`self` 在 `call` 方法体中就是这个值。

让我们来看看标准库中另一个非常常见且有用的枚举：`Option`。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### `Option` 枚举（The `Option` Enum）

本节对 `Option` 进行案例研究，它是标准库定义的另一个枚举。
`Option` 类型编码了这样一种非常常见的场景：某个值可能是某个东西，也可能什么都不是。

例如，如果你请求一个非空列表（List）中的第一个项，你会得到一个值。
如果你请求一个空列表中的第一个项，你会什么也得不到。用类型系统来表达这个概念意味着编译器可以检查你是否处理了所有应该处理的情况；
这一功能可以防止在其他编程语言中极其常见的错误。

编程语言的设计通常被考虑为包含哪些特性，但排除哪些特性也同样重要。
Rust 没有许多其他语言拥有的 null 特性。*Null* 是一个表示没有值存在的值。
在有 null 的语言中，变量总是可能处于两种状态之一：null 或非 null。

在 2009 年的演讲"Null References: The Billion Dollar Mistake"（空引用：十亿美元的错误）中，null 的发明者 Tony Hoare 这样说：

> 我将其称为我的十亿美元错误。那时，我正在为一个面向对象语言设计第一个全面的引用类型系统。
> 我的目标是确保所有引用的使用都是绝对安全的，由编译器自动进行检查。
> 但我无法抗拒加入空引用的诱惑，仅仅因为它实现起来太容易了。
> 这导致了无数的错误、漏洞和系统崩溃，在过去四十年中可能造成了价值十亿美元的痛苦和损失。

null 值的问题在于，如果你尝试将 null 值作为非 null 值使用，你会得到某种错误。
由于这种 null 或非 null 的特性无处不在，因此极易犯此类错误。

然而，null 试图表达的概念仍然是有用的：null 是一个因某种原因当前无效或缺失的值。

问题实际上不在于概念，而在于具体的实现。因此，Rust 没有 null，
但它确实有一个可以编码值存在或缺失概念的枚举。这个枚举就是 `Option<T>`，
它由[标准库定义][option]<!-- ignore -->如下：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` 枚举非常有用，以至于它甚至被包含在预导入（Prelude）中；你无需显式地将其引入作用域。
它的变体也包含在预导入中：你可以直接使用 `Some` 和 `None` 而无需 `Option::` 前缀。
`Option<T>` 枚举仍然只是一个普通的枚举，`Some(T)` 和 `None` 仍然是类型 `Option<T>` 的变体。

`<T>` 语法是 Rust 的一个我们尚未讨论过的特性。它是一个泛型类型参数（Generic Type Parameter），
我们将在第 10 章中更详细地介绍泛型（Generic）。目前，你只需要知道 `<T>` 表示 `Option` 枚举的 `Some` 变体可以持有任意类型的一个数据，
而每个用于替代 `T` 的具体类型都会使整个 `Option<T>` 类型成为一个不同的类型。
以下是一些使用 `Option` 值来持有数字类型和字符类型的示例：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

`some_number` 的类型是 `Option<i32>`。`some_char` 的类型是 `Option<char>`，这是一个不同的类型。
Rust 可以推断出这些类型，因为我们在 `Some` 变体中指定了值。对于 `absent_number`，
Rust 要求我们标注整个 `Option` 类型：编译器无法仅通过查看 `None` 值来推断相应的 `Some` 变体将持有哪种类型。
在这里，我们告诉 Rust 我们打算让 `absent_number` 的类型为 `Option<i32>`。

当我们有一个 `Some` 值时，我们知道存在一个值，并且该值保存在 `Some` 中。
当我们有一个 `None` 值时，从某种意义上说，它与 null 含义相同：我们没有有效值。
那么，为什么拥有 `Option<T>` 比拥有 null 更好呢？

简而言之，因为 `Option<T>` 和 `T`（其中 `T` 可以是任何类型）是不同的类型，编译器不会允许我们将 `Option<T>` 值当作肯定有效的值来使用。
例如，这段代码无法编译，因为它试图将 `i8` 与 `Option<i8>` 相加：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

如果我们运行以上代码的话，那么我们竟会得到以下错误:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

实际上，这个错误消息意味着 Rust 不知道如何将 `i8` 和 `Option<i8>` 相加，因为它们是不同的类型。
当我们在 Rust 中有一个像 `i8` 这样的类型的值时，编译器将确保我们始终有一个有效的值。
我们可以放心地继续使用，而不必在使用该值之前检查 null。
只有当我们有一个 `Option<i8>`（或者我们正在使用的任何类型的值）时，我们才需要担心可能没有值，
并且编译器会确保我们在使用该值之前处理了这种情况。

换句话说，你必须先将 `Option<T>` 转换为 `T`，然后才能对其执行 `T` 的操作。
通常，这有助于捕获 null 最常见的问题之一：假设某个东西不为 null，但实际上它是。

消除错误假定非 null 值的风险有助于你对自己的代码更有信心。
为了拥有一个可能为 null 的值，你必须通过将该值的类型设为 `Option<T>` 来显式选择加入。
然后，在使用该值时，你必须显式处理该值为 null 的情况。
只要一个值的类型不是 `Option<T>`，你*就可以*安全地假定该值不为 null。
这是 Rust 的一个刻意设计决策，旨在限制 null 的普遍性，并增强 Rust 代码的安全性。

那么，当你有一个类型为 `Option<T>` 的值时，如何从 `Some` 变体中获取 `T` 值以便使用该值呢？
`Option<T>` 枚举拥有大量在各种情况下都有用的方法；你可以在[其文档][docs]<!-- ignore -->中查看。
熟悉 `Option<T>` 上的方法将对你的 Rust 之旅极为有益。

一般而言，为了使用 `Option<T>` 值，你需要有处理每个变体的代码。
你需要一些仅在拥有 `Some(T)` 值时才会运行的代码，并且该代码可以使用内部的 `T`。
你需要另一些仅在拥有 `None` 值时才会运行的代码，而该代码没有可用的 `T` 值。
`match` 表达式是一个与枚举一起使用时就恰好能做到这一点的控制流构造（Control Flow Construct）：
它将根据枚举的变体来运行不同的代码，并且这些代码可以使用匹配值内部的数据。

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
