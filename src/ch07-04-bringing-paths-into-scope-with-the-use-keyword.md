## 使用 `use` 关键字将路径引入作用域（Bringing Paths into Scope with the `use` Keyword）

必须写出完整的路径来调用函数可能会让人感到不便和重复。在示例 7-7 中，无论我们选择绝对路径还是相对路径来访问 `add_to_waitlist` 函数，每次我们想要调用 `add_to_waitlist` 时，都必须同时指定 `front_of_house` 和 `hosting`。幸运的是，有一种简化此过程的方法：我们可以使用 `use` 关键字一次性创建一条路径的快捷方式，然后在作用域中的其他地方使用较短的名称。

在示例 7-11 中，我们将 `crate::front_of_house::hosting` 模块引入 `eat_at_restaurant` 函数的作用域中，这样我们只需指定 `hosting::add_to_waitlist` 即可在 `eat_at_restaurant` 中调用 `add_to_waitlist` 函数。

<Listing number="7-11" file-name="src/lib.rs" caption="使用 `use` 将模块引入作用域">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

在作用域中添加 `use` 和路径，类似于在文件系统中创建符号链接（Symbolic Link）。通过在 crate 根中添加 `use crate::front_of_house::hosting`，`hosting` 现在是该作用域中的有效名称，就好像 `hosting` 模块已定义在 crate 根中一样。通过 `use` 引入作用域的路径也会像其他任何路径一样检查私有性。

请注意，`use` 只为其所在的特定作用域创建快捷方式。示例 7-12 将 `eat_at_restaurant` 函数移动到一个名为 `customer` 的新子模块中，该模块与 `use` 语句处于不同的作用域，因此函数体将无法编译。

<Listing number="7-12" file-name="src/lib.rs" caption="`use` 语句仅适用于其所在的作用域">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

编译器错误表明该快捷方式不再适用于 `customer` 模块：

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

请注意，还有一个警告表明 `use` 在其作用域中不再被使用！要解决此问题，请将 `use` 也移动到 `customer` 模块中，或者在子模块 `customer` 中使用 `super::hosting` 引用父模块中的快捷方式。

### 创建惯用的 `use` 路径（Creating Idiomatic `use` Paths）

在示例 7-11 中，你可能想知道为什么我们指定 `use crate::front_of_house::hosting`，然后在 `eat_at_restaurant` 中调用 `hosting::add_to_waitlist`，而不是将 `use` 路径一直指定到 `add_to_waitlist` 函数以实现相同的结果，如示例 7-13 所示。

<Listing number="7-13" file-name="src/lib.rs" caption="使用 `use` 将 `add_to_waitlist` 函数引入作用域，这是不惯用的（unidiomatic）做法">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

虽然示例 7-11 和示例 7-13 完成了相同的任务，但示例 7-11 是使用 `use` 将函数引入作用域的惯用（Idiomatic）方式。使用 `use` 将函数的父模块引入作用域，意味着我们在调用函数时必须指定父模块。在调用函数时指定父模块，清楚地表明该函数不是本地定义的，同时最大限度地减少了对完整路径的重复。而示例 7-13 中的代码则不清楚 `add_to_waitlist` 是在哪里定义的。

另一方面，当使用 `use` 引入结构体（Struct）、枚举（Enum）和其他项时，指定完整路径是惯用的做法。示例 7-14 展示了将标准库的 `HashMap` 结构体引入二进制 crate 作用域的惯用方式。

<Listing number="7-14" file-name="src/main.rs" caption="以惯用方式将 `HashMap` 引入作用域">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

这种惯用做法背后没有强有力的理由：它只是一种逐渐形成的约定，人们已经习惯了以这种方式阅读和编写 Rust 代码。

这种惯用做法的例外是，如果我们使用 `use` 语句将两个同名的项引入作用域，因为 Rust 不允许这样做。示例 7-15 展示了如何将两个具有相同名称但父模块不同的 `Result` 类型引入作用域，以及如何引用它们。

<Listing number="7-15" file-name="src/lib.rs" caption="将两个同名的类型引入相同的作用域需要使用它们的父模块">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

如你所见，使用父模块可以区分两个 `Result` 类型。如果我们改为指定 `use std::fmt::Result` 和 `use std::io::Result`，我们将在同一作用域中有两个 `Result` 类型，当我们使用 `Result` 时，Rust 将不知道我们指的是哪一个。

### 使用 `as` 关键字提供新名称（Providing New Names with the `as` Keyword）

使用 `use` 将两个同名的类型引入同一作用域时，还有另一种解决方案：在路径之后，我们可以指定 `as` 和该类型的一个新的本地名称，即**别名**（Alias）。示例 7-16 展示了另一种编写示例 7-15 中代码的方式，通过使用 `as` 重命名两个 `Result` 类型中的一个。

<Listing number="7-16" file-name="src/lib.rs" caption="使用 `as` 关键字在类型引入作用域时对其进行重命名">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

在第二个 `use` 语句中，我们为 `std::io::Result` 类型选择了新名称 `IoResult`，这不会与我们从 `std::fmt` 引入作用域的 `Result` 产生冲突。示例 7-15 和示例 7-16 都被认为是惯用的，因此你可以自行选择！

### 使用 `pub use` 重新导出名称（Re-exporting Names with `pub use`）

当我们使用 `use` 关键字将名称引入作用域时，该名称在我们导入它的作用域中是私有的。为了使该作用域之外的代码能够引用该名称，就好像它已定义在该作用域中一样，我们可以结合使用 `pub` 和 `use`。这种技术称为**重新导出**（Re-exporting），因为我们将一个项引入作用域，同时也使该项可供其他代码引入它们的作用域。

示例 7-17 展示了示例 7-11 中的代码，其中根模块中的 `use` 已更改为 `pub use`。

<Listing number="7-17" file-name="src/lib.rs" caption="使用 `pub use` 使某个名称可由任何代码从新作用域中使用">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

在此更改之前，外部代码必须使用路径 `restaurant::front_of_house::hosting::add_to_waitlist()` 来调用 `add_to_waitlist` 函数，这还需要将 `front_of_house` 模块标记为 `pub`。现在，由于此 `pub use` 已从根模块重新导出了 `hosting` 模块，外部代码可以改用路径 `restaurant::hosting::add_to_waitlist()`。

当代码的内部结构与调用你的代码的程序员对领域的思考方式不同时，重新导出非常有用。例如，在这个餐厅比喻中，经营餐厅的人会想到"前厅"和"后厨"。但光顾餐厅的顾客可能不会以这些术语来看待餐厅的各个部分。通过 `pub use`，我们可以用一种结构编写代码，但暴露另一种结构。这样做使得我们的库对编写库的程序员和调用库的程序员来说都组织得井井有条。我们将在第 14 章的["导出便捷的公共 API"][ch14-pub-use]<!-- ignore -->中再看一个 `pub use` 的示例，以及它如何影响 crate 的文档。

### 使用外部包（Using External Packages）

在第 2 章中，我们编写了一个猜谜游戏项目，该项目使用了一个名为 `rand` 的外部包（External Package）来获取随机数。为了在我们的项目中使用 `rand`，我们在 _Cargo.toml_ 中添加了以下行：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

在 _Cargo.toml_ 中将 `rand` 添加为依赖项（Dependency）会告诉 Cargo 从 [crates.io](https://crates.io/) 下载 `rand` 包及其任何依赖项，并使 `rand` 可用于我们的项目。

然后，为了将 `rand` 定义引入我们包的作用域，我们添加了一个以 crate 名称 `rand` 开头的 `use` 行，并列出了我们想要引入作用域的项。回想一下，在第 2 章的["生成随机数"][rand]<!-- ignore -->中，我们将 `Rng` 特质（Trait）引入作用域，并调用了 `rand::thread_rng` 函数：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Rust 社区的成员在 [crates.io](https://crates.io/) 上提供了许多包，将其中任何一个引入你的包都涉及相同的步骤：在你的包的 _Cargo.toml_ 文件中列出它们，并使用 `use` 将其 crate 中的项引入作用域。

请注意，标准库 `std` 也是一个相对于我们包的外部 crate。因为标准库随 Rust 语言一起发布，我们不需要更改 _Cargo.toml_ 来包含 `std`。但我们确实需要使用 `use` 来引用它，以将那里的项引入我们包的作用域。例如，对于 `HashMap`，我们将使用以下行：

```rust
use std::collections::HashMap;
```

这是一个以标准库 crate 名称 `std` 开头的绝对路径。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### 使用嵌套路径整理 `use` 列表（Using Nested Paths to Clean Up `use` Lists）

如果我们使用定义在同一个 crate 或同一个模块中的多个项，将每个项列在单独的一行上会占用文件中大量的垂直空间。例如，我们在示例 2-4 的猜谜游戏中的这两个 `use` 语句将项从 `std` 引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

相反，我们可以使用嵌套路径（Nested Path）将相同的项在一行中引入作用域。我们通过指定路径的公共部分，后跟两个冒号，然后在大括号中列出路径中不同的部分来实现，如示例 7-18 所示。

<Listing number="7-18" file-name="src/main.rs" caption="指定嵌套路径以将具有相同前缀的多个项引入作用域">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

在较大的程序中，使用嵌套路径将许多来自相同 crate 或模块的项引入作用域，可以大大减少所需的单独 `use` 语句数量！

我们可以在路径的任何级别使用嵌套路径，这在组合两个共享子路径的 `use` 语句时非常有用。例如，示例 7-19 展示了两个 `use` 语句：一个将 `std::io` 引入作用域，另一个将 `std::io::Write` 引入作用域。

<Listing number="7-19" file-name="src/lib.rs" caption="两个 `use` 语句，其中一个路径是另一个的子路径">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

这两个路径的公共部分是 `std::io`，这也是第一个路径的完整内容。要将这两个路径合并为一个 `use` 语句，我们可以在嵌套路径中使用 `self`，如示例 7-20 所示。

<Listing number="7-20" file-name="src/lib.rs" caption="将示例 7-19 中的路径合并为一个 `use` 语句">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

此行将 `std::io` 和 `std::io::Write` 引入作用域。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### 使用全局运算符导入项（Importing Items with the Glob Operator）

如果我们想将某个路径中定义的**所有**公共项引入作用域，可以指定该路径后跟 `*` 全局运算符（Glob Operator）：

```rust
use std::collections::*;
```

这个 `use` 语句将 `std::collections` 中定义的所有公共项引入当前作用域。使用全局运算符时要小心！全局运算符可能会使人更难分辨哪些名称在作用域中，以及程序中使用的某个名称是在哪里定义的。此外，如果依赖项更改了其定义，你导入的内容也会随之改变，这可能会在升级依赖项时导致编译器错误，例如依赖项添加了一个与你在同一作用域中的定义同名的定义。

全局运算符通常用于测试，将测试对象的所有内容引入 `tests` 模块；我们将在第 11 章的["如何编写测试"][writing-tests]<!-- ignore -->中讨论这一点。全局运算符有时也作为预导入模式（Prelude Pattern）的一部分使用：请参阅[标准库文档](../std/prelude/index.html#other-preludes)<!-- ignore -->了解更多关于该模式的信息。

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
