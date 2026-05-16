<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## `match` 控制流构造（The `match` Control Flow Construct）

Rust 有一个极其强大的控制流构造（Control Flow Construct）称为 `match`，它允许你将一个值与一系列模式（Pattern）进行比较，然后根据匹配的模式执行代码。
模式可以由字面量值（Literal Value）、变量名、通配符（Wildcard）和许多其他内容组成；
[第 19 章][ch19-00-patterns]<!-- ignore -->涵盖了所有不同类型的模式及其功能。
`match` 的强大之处在于模式的表现力以及编译器会确认所有可能的情况都得到了处理。

将 `match` 表达式想象成一台硬币分拣机：硬币沿着带有各种大小孔洞的轨道滑下，每枚硬币都会从它遇到的第一个适合的孔洞中落下。
类似地，值会依次通过 `match` 中的每个模式，在值"适合"的第一个模式处，该值落入关联的代码块中并在执行期间被使用。

说到硬币，让我们用 `match` 以它们为例！我们可以编写一个函数，接收一枚未知的美国硬币，
以类似于计数机器的方式确定它是哪种硬币，并返回其面值（以美分为单位），如示例 6-3 所示。

<Listing number="6-3" caption="一个枚举和一个以枚举的变体作为模式的 `match` 表达式">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

让我们分解 `value_in_cents` 函数中的 `match`。首先，我们列出 `match` 关键字，后跟一个表达式，
在本例中是值 `coin`。这看起来与 `if` 使用的条件表达式非常相似，但有一个很大的区别：
对于 `if`，条件需要求值为一个布尔值（Boolean），但在这里它可以是任何类型。
本例中 `coin` 的类型是我们在第一行定义的 `Coin` 枚举。

接下来是 `match` 分支（Arm）。一个分支有两个部分：一个模式和一些代码。
这里的第一个分支有一个模式，即值 `Coin::Penny`，然后是 `=>` 运算符，它将模式和要运行的代码分隔开。
本例中的代码就是值 `1`。每个分支用逗号与下一个分支分隔。

当 `match` 表达式执行时，它会将结果值按顺序与每个分支的模式进行比较。如果一个模式匹配该值，则执行与该模式关联的代码。如果该模式不匹配该值，则继续执行下一个分支，这与硬币分拣机非常相似。我们可以根据需要拥有任意多个分支：在示例 6-3 中，我们的 `match` 有四个分支。

每个分支关联的代码都是一个表达式（Expression），而匹配分支中表达式的结果值就是整个 `match` 表达式返回的值。

如果 match 分支代码很短，我们通常不会使用花括号，就像示例 6-3 中每个分支只是返回一个值那样。如果你想在 match 分支中运行多行代码，你必须使用花括号，并且分支后面的逗号是可选的。例如，以下代码在每次使用 `Coin::Penny` 调用该方法时都会打印“Lucky penny!”，但它仍然返回块的最后一个值 `1`：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### 绑定值的模式（Patterns That Bind to Values）

`match` 分支的另一个有用特性是它们可以绑定到与模式匹配的值的一部分。这就是我们可以从枚举变体中提取值的方式。

举个例子，让我们修改一个枚举变体，使其内部包含数据。从 1999 年到 2008 年，美国铸造的 25 美分硬币（quarters）在其中一个面印有 50 个州中每个州的不同图案。没有其他硬币拥有州图案，因此只有 25 美分硬币有这个额外值。我们可以通过将 `Quarter` 变体修改为包含一个存储在其中的 `UsState` 值来将这些信息添加到我们的 `enum` 中，我们在示例 6-4 中已经这样做了。

<Listing number="6-4" caption="一个 `Coin` 枚举，其中 `Quarter` 变体还持有一个 `UsState` 值">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

假设一个朋友正在努力收集全部 50 个州的 25 美分硬币。当我们按硬币类型分类零钱时，我们还会喊出每个 25 美分硬币关联的州名，这样如果朋友还没有这个州的硬币，他们就可以将其添加到收藏中。

在此代码的 match 表达式中，我们向匹配 `Coin::Quarter` 变体值的模式添加了一个名为 `state` 的变量。当匹配到 `Coin::Quarter` 时，`state` 变量将绑定到该 25 美分硬币的州的值。然后，我们可以在该分支的代码中使用 `state`，如下所示：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

如果我们调用 `value_in_cents(Coin::Quarter(UsState::Alaska))`，`coin` 将是 `Coin::Quarter(UsState::Alaska)`。当我们将该值与每个 match 分支进行比较时，直到 `Coin::Quarter(state)` 之前没有任何分支匹配。此时，`state` 的绑定将是值 `UsState::Alaska`。然后我们可以在 `println!` 表达式中使用该绑定，从而从 `Quarter` 的 `Coin` 枚举变体中取出内部的州值。

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### `Option<T>` 的 `match` 模式（The `Option<T>` `match` Pattern）


在上一节中，我们想要在使用 `Option<T>` 时从 `Some` 中取出内部的 `T` 值；我们也可以使用 `match` 来处理 `Option<T>`，就像我们对 `Coin` 枚举所做的那样！不再比较硬币，而是比较 `Option<T>` 的变体，但 `match` 表达式的工作方式保持不变。

假设我们想编写一个函数，它接收一个 `Option<i32>`，如果内部有值，则将该值加 1。如果内部没有值，则函数应返回 `None` 值，并且不尝试执行任何操作。

得益于 `match`，这个函数非常容易编写，如示例 6-5 所示。

<Listing number="6-5" caption="一个在 `Option<i32>` 上使用 `match` 表达式的函数">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

让我们更详细地检查 `plus_one` 的第一次执行。当我们调用 `plus_one(five)` 时，`plus_one` 函数体中的变量 `x` 将具有值 `Some(5)`。然后我们将该值与每个 match 分支进行比较：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` 值不匹配模式 `None`，因此我们继续到下一个分支：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` 匹配 `Some(i)` 吗？匹配了！我们有相同的变体。`i` 绑定到 `Some` 中包含的值，因此 `i` 的值为 `5`。然后执行 match 分支中的代码，因此我们将 `i` 的值加 1，并创建一个新的 `Some` 值，其中包含我们的总数 `6`。

现在让我们考虑示例 6-5 中 `plus_one` 的第二次调用，其中 `x` 是 `None`。我们进入 `match` 并与第一个分支进行比较：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

匹配了！没有要加的值，因此程序停止并返回 `=>` 右侧的 `None` 值。因为第一个分支已匹配，不再比较其他分支。

结合使用 `match` 和枚举在许多情况下都很有用。你会在 Rust 代码中经常看到这种模式：对枚举进行 `match`，将变量绑定到内部的数据，然后基于它执行代码。一开始可能有点棘手，但一旦你习惯了，你会希望所有语言都有这个特性。它一直是用户的最爱。

### `match` 是穷尽的（Matches Are Exhaustive）

关于 `match` 还有一个方面需要讨论：分支的模式必须覆盖所有可能性。考虑我们 `plus_one` 函数的这个版本，它包含一个 bug，无法编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

我们没有处理 `None` 的情况，因此这段代码会导致 bug。幸运的是，这是 Rust 能够捕捉到的 bug。如果我们尝试编译这段代码，会得到以下错误：

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust 知道我们没有覆盖所有可能的情况，甚至知道我们忘记了哪个模式！Rust 中的 `match` 是**穷尽的（exhaustive）**：我们必须覆盖每一种可能性，代码才能有效。特别是在 `Option<T>` 的情况下，当 Rust 阻止我们忘记显式处理 `None` 情况时，它保护了我们不会在可能为 null 时假设我们有一个值，从而使之前讨论的十亿美元错误（billion-dollar mistake）变得不可能。

### 通配模式与 `_` 占位符（Catch-All Patterns and the `_` Placeholder）

使用枚举，我们还可以对少数特定值执行特殊操作，而对所有其他值执行一个默认操作。假设我们在实现一个游戏，如果掷骰子掷出 3，你的角色不会移动，而是获得一顶漂亮的新帽子。如果掷出 7，你的角色会失去一顶漂亮的帽子。对于所有其他值，你的角色在游戏板上移动相应数量的格子。下面是一个实现该逻辑的 `match` 表达式，掷骰子的结果是硬编码的而不是随机值，所有其他逻辑由没有函数体的函数表示，因为实际实现它们超出了本例的范围：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

对于前两个分支，模式是字面量值 `3` 和 `7`。对于覆盖所有其他可能值的最后一个分支，模式是我们选择命名为 `other` 的变量。`other` 分支运行的代码通过将该变量传递给 `move_player` 函数来使用它。

这段代码可以编译，尽管我们还没有列出 `u8` 可以拥有的所有可能值，因为最后一个模式将匹配所有未特别列出的值。这种通配模式满足了 `match` 必须穷尽的要求。请注意，我们必须将通配分支放在最后，因为模式是按顺序求值的。如果我们把通配分支放在前面，其他分支将永远不会运行，因此如果我们添加位于通配分支之后的分支，Rust 会发出警告！

Rust 也有一种模式，当我们想要通配但不想在通配模式中**使用**该值时可以使用：`_` 是一个特殊模式，它匹配任何值但不会绑定到该值。这告诉 Rust 我们不打算使用该值，因此 Rust 不会警告我们有一个未使用的变量。

让我们修改游戏规则：现在，如果你掷出的数字不是 3 或 7，你必须重新掷。我们不再需要使用通配值，因此我们可以修改代码，使用 `_` 代替名为 `other` 的变量：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

这个示例也满足了穷尽性要求，因为我们在最后一个分支中显式地忽略了所有其他值；我们没有忘记任何东西。

最后，我们再次修改游戏规则，这样如果你掷出的数字不是 3 或 7，你的回合就不会发生任何事情。我们可以通过使用单元值（unit value）（即我们在[“元组类型（The Tuple Type）”][tuples]<!-- ignore -->章节中提到的空元组类型）作为 `_` 分支的代码来表达这一点：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

在这里，我们显式地告诉 Rust，我们不打算使用任何不匹配前面分支模式的其他值，并且我们不想在这种情况下运行任何代码。

关于模式和匹配的更多内容，我们将在[第 19 章][ch19-00-patterns]<!-- ignore -->中介绍。现在，我们将继续学习 `if let` 语法，它在 `match` 表达式略显冗长的情况下可能很有用。

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
