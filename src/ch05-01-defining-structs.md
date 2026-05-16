## 定义和实例化结构体（Struct）

结构体（Struct）与[“元组类型”][tuples]<!-- ignore -->部分讨论的元组（Tuple）类似，两者都包含多个相关的值。与元组一样，结构体的各个部分可以是不同的类型。与元组不同的是，在结构体中你需要为每一段数据命名，以便清楚地表明这些值的含义。添加这些名称意味着结构体比元组更灵活：你不必依赖数据的顺序来指定或访问实例的值。

要定义一个结构体（Struct），我们输入关键字 `struct` 并为整个结构体命名。结构体的名称应该描述被组合在一起的数据片段的意义。然后，在花括号内，我们定义数据片段的名称和类型，我们称之为*字段（Field）*。例如，示例 5-1 展示了一个存储用户账户信息的结构体。

<Listing number="5-1" file-name="src/main.rs" caption="一个 `User` 结构体定义">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

在定义结构体之后，我们通过为每个字段指定具体的值来创建该结构体的*实例（Instance）*。我们通过写出结构体的名称，然后添加包含 `key: value` 对的花括号来创建实例，其中键是字段的名称，值是我们想要在这些字段中存储的数据。我们不必按照在结构体中声明字段时的相同顺序来指定字段。换句话说，结构体定义就像是该类型的通用模板，而实例则用特定的数据填充该模板来创建该类型的值。例如，我们可以像示例 5-2 那样声明一个特定的用户。

<Listing number="5-2" file-name="src/main.rs" caption="Creating an instance of the `User` struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

要从结构体中获取特定值，我们使用点号（Dot）表示法。例如，要访问这个用户的电子邮件地址，我们使用 `user1.email`。如果实例是可变的，我们可以使用点号表示法并赋值给特定字段来改变值。示例 5-3 展示了如何更改可变 `User` 实例中 `email` 字段的值。

<Listing number="5-3" file-name="src/main.rs" caption="更改 `User` 实例的 `email` 字段的值">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

注意，整个实例必须是可变的；Rust 不允许我们只将某些字段标记为可变。与任何表达式一样，我们可以将结构体的新实例构造为函数体中的最后一个表达式，以隐式返回该新实例。

示例 5-4 展示了一个 `build_user` 函数，它接收给定的 email 和 username 并返回一个 `User` 实例。`active` 字段的值为 `true`，`sign_in_count` 的值为 `1`。

<Listing number="5-4" file-name="src/main.rs" caption="一个 `build_user` 函数，它接收 email 和 username 并返回一个 `User` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

将函数参数命名为与结构体字段相同的名称是有意义的，但必须重复 `email` 和 `username` 字段名称和变量有点繁琐。如果结构体有更多字段，重复每个名称会更加烦人。幸运的是，有一个便捷的简写！

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### 使用字段初始化简写（Field Init Shorthand）

因为在示例 5-4 中参数名称和结构体字段名称完全相同，我们可以使用*字段初始化简写（Field Init Shorthand）*语法重写 `build_user`，使其行为完全相同，但不需要重复 `username` 和 `email`，如示例 5-5 所示。

<Listing number="5-5" file-name="src/main.rs" caption="一个使用字段初始化简写的 `build_user` 函数，因为 `username` 和 `email` 参数与结构体字段同名">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

这里，我们创建了一个新的 `User` 结构体实例，它有一个名为 `email` 的字段。我们想将 `email` 字段的值设置为 `build_user` 函数的 `email` 参数的值。因为 `email` 字段和 `email` 参数同名，我们只需要写 `email` 而不是 `email: email`。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### 使用结构体更新语法（Struct Update Syntax）创建实例

通常，创建一个新结构体实例是很有用的，它包含另一个同类型实例的大部分值，但改变其中的一部分。你可以使用结构体更新语法（Struct Update Syntax）来实现这一点。

首先，在示例 5-6 中，我们展示了如何以常规方式（不使用更新语法）在 `user2` 中创建一个新的 `User` 实例。我们为 `email` 设置了一个新值，但其他值使用了我们在示例 5-2 中创建的 `user1` 中的值。

<Listing number="5-6" file-name="src/main.rs" caption="使用除一个值以外的所有来自 `user1` 的值创建一个新的 `User` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

使用结构体更新语法（Struct Update Syntax），我们可以用更少的代码达到相同的效果，如示例 5-7 所示。`..` 语法指定了未显式设置的其余字段应与给定实例中的字段具有相同的值。

<Listing number="5-7" file-name="src/main.rs" caption="使用结构体更新语法为 `User` 实例设置新的 `email` 值，但使用 `user1` 的其余值">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

示例 5-7 中的代码也在 `user2` 中创建了一个实例，它的 `email` 值不同，但 `username`、`active` 和 `sign_in_count` 字段的值与 `user1` 相同。`..user1` 必须放在最后，以指定任何剩余字段应从 `user1` 的对应字段中获取值，但我们可以选择以任何顺序为任意多个字段指定值，而不受结构体定义中字段顺序的限制。

注意，结构体更新语法（Struct Update Syntax）使用 `=` 就像赋值一样；这是因为它会移动数据，正如我们在["变量与数据交互的方式：移动"][move]<!-- ignore -->部分看到的那样。在这个例子中，创建 `user2` 后我们不能再使用 `user1`，因为 `user1` 的 `username` 字段中的 `String` 被移动到了 `user2` 中。如果我们为 `user2` 的 `email` 和 `username` 都提供了新的 `String` 值，从而只使用了 `user1` 中的 `active` 和 `sign_in_count` 值，那么 `user2` 创建后 `user1` 仍然有效。`active` 和 `sign_in_count` 都是实现了 `Copy` trait 的类型，因此我们在["栈（Stack）上的数据：Copy"][copy]<!-- ignore -->部分讨论的行为将适用。在这个例子中，我们仍然可以使用 `user1.email`，因为它的值没有被移出 `user1`。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### 使用元组结构体（Tuple Struct）创建不同类型

Rust 还支持看起来类似于元组（Tuple）的结构体，称为*元组结构体（Tuple Struct）*。元组结构体具有结构体名称带来的额外含义，但与其字段没有关联的名称；相反，它们只有字段的类型。当你想要给整个元组一个名称并使该元组成为与其他元组不同的类型时，以及当像常规结构体那样命名每个字段会显得冗长或多余时，元组结构体（Tuple Struct）非常有用。

要定义一个元组结构体（Tuple Struct），以 `struct` 关键字和结构体名称开头，后跟元组中的类型。例如，这里我们定义并使用了两个名为 `Color` 和 `Point` 的元组结构体：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

注意，`black` 和 `origin` 的值是不同类型，因为它们是不同元组结构体（Tuple Struct）的实例。你定义的每个结构体都是自己的类型，即使结构体内的字段可能具有相同的类型。例如，一个接受 `Color` 类型参数的函数不能接受 `Point` 作为参数，即使这两种类型都是由三个 `i32` 值组成的。除此之外，元组结构体实例与元组类似，你可以将它们解构为各个独立的部分，也可以使用 `.` 后跟索引来访问单个值。与元组不同，元组结构体在解构时需要你指定结构体的类型。例如，我们写 `let Point(x, y, z) = origin;` 来将 `origin` 点中的值解构为名为 `x`、`y` 和 `z` 的变量。

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### 定义类单元结构体（Unit-Like Struct）

你还可以定义没有任何字段的结构体！这些被称为*类单元结构体（Unit-Like Struct）*，因为它们的行为类似于 `()`（我们在["元组类型"][tuples]<!-- ignore -->部分提到的单元类型）。当你需要在某个类型上实现 trait 但又不希望在该类型本身中存储任何数据时，类单元结构体（Unit-Like Struct）非常有用。我们将在第 10 章讨论 trait。下面是一个声明和实例化名为 `AlwaysEqual` 的类单元结构体的例子：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

要定义 `AlwaysEqual`，我们使用 `struct` 关键字，后跟我们想要的名称，然后是一个分号。不需要花括号或圆括号！然后，我们可以用类似的方式在 `subject` 变量中获取 `AlwaysEqual` 的实例：使用我们定义的名称，不需要任何花括号或圆括号。想象一下，稍后我们将为这个类型实现行为，使得 `AlwaysEqual` 的每个实例总是与任何其他类型的每个实例相等，也许是出于测试目的而需要一个已知的结果。我们不需要任何数据来实现那个行为！你将在第 10 章中看到如何定义 trait 并在任何类型（包括类单元结构体）上实现它们。

> ### Ownership of Struct Data
>
> In the `User` struct definition in Listing 5-1, we used the owned `String`
> type rather than the `&str` string slice type. This is a deliberate choice
> because we want each instance of this struct to own all of its data and for
> that data to be valid for as long as the entire struct is valid.
>
> 结构体也可以存储对其他对象所有数据的引用，但要做到这一点需要使用*生命周期（Lifetime）*，这是我们在第 10 章将要讨论的一个 Rust 特性。生命周期（Lifetime）确保结构体引用的数据在结构体存在的期间内始终有效。假设你尝试在结构体中存储引用而不指定生命周期，就像下面 *src/main.rs* 中的这样；这不会工作：
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> 编译器会报错，提示需要生命周期（Lifetime）说明符：
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> 在第 10 章中，我们将讨论如何修复这些错误，以便你可以在结构体中存储引用，但现在，我们将使用像 `String` 这样的拥有所有权的类型而不是像 `&str` 这样的引用来修复此类错误。

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
