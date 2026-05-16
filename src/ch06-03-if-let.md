## 使用 `if let` 和 `let...else` 的简洁控制流（Concise Control Flow with `if let` and `let...else`）

`if let` 语法让你可以将 `if` 和 `let` 组合成一种更简洁的方式来处理匹配一种模式的值，同时忽略其余值。考虑示例 6-6 中的程序，它对 `config_max` 变量中的 `Option<u8>` 值进行匹配，但只在值为 `Some` 变体时才执行代码。

<Listing number="6-6" caption="一个只关心在值为 `Some` 时执行代码的 `match` 表达式">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

如果值是 `Some`，我们通过在模式中将值绑定到变量 `max` 来打印出 `Some` 变体中的值。我们不想对 `None` 值做任何处理。为了满足 `match` 表达式的要求，我们在只处理了一个变体之后必须加上 `_ => ()`，这是烦人的样板代码（Boilerplate Code）。

相反，我们可以使用 `if let` 以更短的方式编写。以下代码的行为与示例 6-6 中的 `match` 相同：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` 语法接受一个模式和一个表达式，两者用等号分隔。它的工作方式与 `match` 相同，其中表达式被传递给 `match`，而模式则是其第一个分支。在这个例子中，模式是 `Some(max)`，`max` 绑定到 `Some` 内部的值。然后，我们可以在 `if let` 块的函数体中使用 `max`，就像在相应的 `match` 分支中使用 `max` 一样。`if let` 块中的代码只在值匹配该模式时才运行。

使用 `if let` 意味着更少的输入、更少的缩进和更少的样板代码。但是，你失去了 `match` 强制要求的穷尽性检查（Exhaustive Checking），该检查能确保你不会忘记处理任何情况。在 `match` 和 `if let` 之间进行选择取决于你在特定情况下的操作，以及简洁性的获得是否值得失去穷尽性检查。

换句话说，你可以将 `if let` 视为 `match` 的语法糖（Syntax Sugar），它在值匹配一个模式时运行代码，然后忽略所有其他值。

我们可以在 `if let` 中包含一个 `else`。与 `else` 配套的代码块等同于与 `if let` 和 `else` 等价的 `match` 表达式中 `_` 分支的代码块。回顾示例 6-4 中的 `Coin` 枚举定义，其中 `Quarter` 变体还持有一个 `UsState` 值。如果我们想统计所有看到的非 25 美分硬币，同时公布 25 美分的州，我们可以使用 `match` 表达式来实现，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

或者我们可以使用 `if let` 和 `else` 表达式，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## 使用 `let...else` 保持在"快乐路径"上（Staying on the "Happy Path" with `let...else`）

一种常见的模式是当值存在时执行某些计算，否则返回默认值。继续我们带有 `UsState` 值的硬币示例，如果我们想根据 25 美分硬币上的州的历史年代说一些有趣的话，我们可能会在 `UsState` 上引入一个方法来检查州的年龄，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

然后，我们可能使用 `if let` 来匹配硬币的类型，在条件体内引入一个 `state` 变量，如示例 6-7 所示。

<Listing number="6-7" caption="通过嵌套在 `if let` 内部的条件语句来检查一个州在 1900 年是否存在">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

这样能完成任务，但它将工作推入了 `if let` 语句的函数体中，如果要完成的工作更复杂，可能很难看清楚顶层分支之间的关系。我们也可以利用表达式会产生值这一事实，要么从 `if let` 产生 `state`，要么提前返回，如示例 6-8 所示。（你也可以用 `match` 实现类似的效果。）

<Listing number="6-8" caption="使用 `if let` 产生一个值或提前返回">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

不过，这种方式也有点难以理解！`if let` 的一个分支产生一个值，而另一个分支则直接从函数中返回。

为了让这种常见模式更易于表达，Rust 提供了 `let...else`。`let...else` 语法在左侧接受一个模式，在右侧接受一个表达式，与 `if let` 非常相似，但它没有 `if` 分支，只有 `else` 分支。如果模式匹配，它会将模式中的值绑定到外部作用域。如果模式**不**匹配，程序将进入 `else` 分支，该分支必须从函数中返回。

在示例 6-9 中，你可以看到使用 `let...else` 代替 `if let` 时示例 6-8 的效果。

<Listing number="6-9" caption="使用 `let...else` 来澄清函数内部的流程">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

注意，通过这种方式，函数的主体保持在"快乐路径"（Happy Path）上，而不像 `if let` 那样两个分支具有明显不同的控制流。

如果你遇到程序逻辑过于冗长而无法用 `match` 表达的情况，请记住 `if let` 和 `let...else` 也在你的 Rust 工具箱中。

## 总结（Summary）

到目前为止，我们已经介绍了如何使用枚举（Enum）来创建自定义类型（Custom Type），这些类型可以是一组枚举值中的一个。我们已经展示了标准库的 `Option<T>` 类型如何帮助你使用类型系统来防止错误。当枚举值包含数据时，你可以使用 `match` 或 `if let` 来提取和使用这些值，具体取决于你需要处理多少种情况。

你的 Rust 程序现在可以使用结构体（Struct）和枚举来表达你所在领域的概念。创建自定义类型在 API 中使用可以确保类型安全：编译器将确保你的函数只获得每个函数所期望的类型的值。

为了向你的用户提供一个组织良好的 API，该 API 使用起来简单明了，并且只暴露你的用户真正需要的内容，现在让我们转向 Rust 的模块（Module）。
