## 方法（Methods）

方法（Method）与函数（Function）类似：我们用 `fn` 关键字和一个名称来声明它们，
它们可以有参数和返回值，并且包含一些在从其他地方调用该方法时运行的代码。
与函数不同的是，方法是在结构体（Struct）（或者枚举（Enum）或 trait 对象（Trait Object））的上下文中定义的，
我们分别在[第 6 章][enums]<!-- ignore -->和[第 18 章][trait-objects]<!-- ignore -->中介绍，
并且它们的第一个参数始终是 `self`，它表示调用该方法的结构体实例。

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### 方法语法（Method Syntax）

让我们将那个以 `Rectangle` 实例作为参数的 `area` 函数，改为一个定义在 `Rectangle` 结构体上的 `area` 方法，
如示例 5-13 所示。

<Listing number="5-13" file-name="src/main.rs" caption="在 `Rectangle` 结构体上定义一个 `area` 方法">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

为了在 `Rectangle` 的上下文中定义这个函数，我们为 `Rectangle` 启动一个 `impl`（implementation，实现）块。
这个 `impl` 块中的所有内容都将与 `Rectangle` 类型相关联。
然后，我们将 `area` 函数移到 `impl` 的花括号内，并将签名和函数体内部的第一个（在这里也是唯一一个）参数改为 `self`。
在 `main` 函数中，我们之前调用 `area` 函数并传入 `rect1` 作为参数的地方，现在可以改用*方法语法（Method Syntax）*来调用 `Rectangle` 实例上的 `area` 方法。
方法语法位于实例之后：我们加上一个点号，后跟方法名称、圆括号以及任何参数。

在 `area` 的签名中，我们使用 `&self` 而不是 `rectangle: &Rectangle`。
`&self` 实际上是 `self: &Self` 的简写。在 `impl` 块中，类型 `Self` 是该 `impl` 块所针对的类型的别名。
方法的第一个参数**必须**是一个名为 `self`、类型为 `Self` 的参数，因此 Rust 允许你在第一个参数位置只用 `self` 这个名字来缩写它。
请注意，我们仍然需要在 `self` 简写前面使用 `&` 来表明该方法借用了 `Self` 实例，就像我们在 `rectangle: &Rectangle` 中所做的那样。
方法可以获取 `self` 的所有权（Ownership），像我们这里所做的那样不可变地借用（Borrow）`self`，或者可变地借用 `self`，
就像它们对其他任何参数所做的那样。

我们在这里选择 `&self` 的原因与在函数版本中我们使用 `&Rectangle` 的原因相同：
我们不希望获取所有权，只是想读取结构体中的数据，而不是写入。如果我们想在该方法所做的事情中更改调用该方法的实例，
我们会使用 `&mut self` 作为第一个参数。让一个方法仅使用 `self` 作为第一个参数来获取实例的所有权是很少见的；
这种技术通常是在该方法将 `self` 转换为其他内容，并且你希望阻止调用方在转换后使用原始实例时使用。

使用方法而不是函数的主要原因，除了提供方法语法和不必在每个方法的签名中重复 `self` 的类型之外，是为了组织。
我们将对某个类型的实例可以做的所有事情都放在一个 `impl` 块中，而不是让我们代码的未来使用者在所提供库的各个地方搜索 `Rectangle` 的功能。

请注意，我们可以选择为方法赋予与结构体字段相同的名称。
例如，我们可以在 `Rectangle` 上定义一个也名为 `width` 的方法：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

在这里，我们选择让 `width` 方法在实例的 `width` 字段的值大于 `0` 时返回 `true`，在值为 `0` 时返回 `false`：
我们可以在任何目的下在同名方法中使用字段。在 `main` 中，当我们在 `rect1.width` 后面跟上括号时，Rust 知道我们指的是方法 `width`。
当不使用括号时，Rust 知道我们指的是字段 `width`。

通常（但并非总是），当我们为方法赋予与字段相同的名称时，我们希望它只返回该字段的值，不做任何其他事情。
像这样的方法称为 *getter 方法（Getter）*，Rust 不会像某些其他语言那样为结构体字段自动实现它们。
Getter 方法很有用，因为你可以将字段设为私有（Private）但方法设为公有（Public），从而作为类型公有 API 的一部分启用对该字段的只读访问。
我们将在[第 7 章][public]<!-- ignore -->中讨论公有和私有的含义，以及如何将字段或方法指定为公有或私有。

> ### `->` 运算符去哪了？（Where's the `->` Operator?）
>
> 在 C 和 C++ 中，调用方法时使用两种不同的运算符：如果你直接在对象上调用方法，使用 `.`；
> 如果你在指向对象的指针上调用方法并需要先解引用（Dereference）指针，则使用 `->`。
> 换句话说，如果 `object` 是一个指针，那么 `object->something()` 类似于 `(*object).something()`。
>
> Rust 没有与 `->` 运算符等效的东西；相反，Rust 有一个称为*自动引用与解引用（Automatic Referencing and Dereferencing）*的功能。
> 调用方法是 Rust 中少数具有此行为的地方之一。
>
> 它的工作原理如下：当你使用 `object.something()` 调用某个方法时，Rust 会自动添加 `&`、`&mut` 或 `*` 使 `object` 与该方法的签名匹配。
> 换句话说，以下两种写法是相同的：
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 第一种写法看起来要干净得多。这种自动引用行为之所以有效，是因为方法有一个明确的接收者（Receiver）——即 `self` 的类型。
> 给定接收者和方法名称，Rust 可以明确地判断出该方法是读取（`&self`）、修改（`&mut self`）还是消费（`self`）。
> Rust 对方法接收者进行隐式借用（Borrow）这一事实，是使所有权在实践中具有良好人机工程学特性的一个重要原因。

### 带有更多参数的方法（Methods with More Parameters）

让我们通过在 `Rectangle` 结构体上实现第二个方法来练习使用方法。
这一次，我们希望 `Rectangle` 的一个实例接收另一个 `Rectangle` 实例作为参数，
如果第二个 `Rectangle` 能够完全放入 `self`（第一个 `Rectangle`）中，则返回 `true`；否则返回 `false`。
也就是说，一旦我们定义了 `can_hold` 方法，我们就能够编写如示例 5-14 所示的程序。

<Listing number="5-14" file-name="src/main.rs" caption="使用尚未编写的 `can_hold` 方法">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

预期的输出如下所示，因为 `rect2` 的两个维度都小于 `rect1` 的维度，但 `rect3` 比 `rect1` 宽：

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

我们知道我们想要定义一个方法，因此它将位于 `impl Rectangle` 块中。
方法名将是 `can_hold`，它将接受另一个 `Rectangle` 的不可变借用（Immutable Borrow）作为参数。
我们可以通过查看调用该方法的代码来判断参数的类型：
`rect1.can_hold(&rect2)` 传入了 `&rect2`，这是对 `rect2`（一个 `Rectangle` 实例）的不可变借用。
这是合理的，因为我们只需要读取 `rect2`（而不是写入，写入意味着我们需要可变借用），
并且我们希望 `main` 保留 `rect2` 的所有权，以便在调用 `can_hold` 方法后可以再次使用它。
`can_hold` 的返回值将是一个布尔值（Boolean），其实现将分别检查 `self` 的宽度和高度是否大于另一个 `Rectangle` 的宽度和高度。
让我们将新的 `can_hold` 方法添加到示例 5-13 中的 `impl` 块中，如示例 5-15 所示。

<Listing number="5-15" file-name="src/main.rs" caption="在 `Rectangle` 上实现 `can_hold` 方法，该方法接收另一个 `Rectangle` 实例作为参数">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

当我们使用示例 5-14 中的 `main` 函数运行此代码时，我们将得到预期的输出。
方法可以接受多个参数，这些参数是在 `self` 参数之后添加到签名中的，并且这些参数的工作方式与函数中的参数一样。

### 关联函数（Associated Functions）

在 `impl` 块中定义的所有函数都称为*关联函数（Associated Functions）*，因为它们与 `impl` 后面命名的类型相关联。
我们可以定义没有 `self` 作为第一个参数的关联函数（因此不是方法），因为它们不需要类型的实例来工作。
我们已经使用过一个这样的函数：定义在 `String` 类型上的 `String::from` 函数。

不是方法的关联函数通常用作构造函数（Constructor），它们将返回该结构体的新实例。
这些通常被称为 `new`，但 `new` 不是一个特殊的名称，也不是内建在语言中的。
例如，我们可以选择提供一个名为 `square` 的关联函数，它只有一个维度参数，并将其同时用作宽度和高度，
这样就能更容易地创建一个正方形 `Rectangle`，而不必指定两次相同的值：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

返回类型和函数体中的 `Self` 关键字是 `impl` 关键字后面出现的类型的别名，在本例中是 `Rectangle`。

要调用这个关联函数，我们使用结构体名称加上 `::` 语法；例如 `let sq = Rectangle::square(3);`。
这个函数被结构体命名空间（Namespace）化：`::` 语法既用于关联函数，也用于模块（Module）创建的命名空间。
我们将在[第 7 章][modules]<!-- ignore -->中讨论模块。

### 多个 `impl` 块（Multiple `impl` Blocks）

每个结构体允许拥有多个 `impl` 块。例如，示例 5-15 等价于示例 5-16 中所示的代码，其中每个方法都有自己的 `impl` 块。

<Listing number="5-16" caption="使用多个 `impl` 块重写示例 5-15">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

在这里，将这些方法分离到多个 `impl` 块中没有任何理由，但这是有效的语法。
我们将在第 10 章中看到多个 `impl` 块有用的情况，届时我们将讨论泛型（Generic）类型和 trait。

## 总结（Summary）

结构体让你可以创建对你的领域有意义的自定义类型（Custom Type）。
通过使用结构体，你可以将相关联的数据片段保持彼此连接，并为每个片段命名，使你的代码清晰明了。
在 `impl` 块中，你可以定义与你的类型相关联的函数，而方法是关联函数的一种，让你可以指定结构体实例所具有的行为。

但结构体并不是创建自定义类型的唯一方式：让我们转向 Rust 的枚举（Enum）功能，为你的工具箱再添一件工具。

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
