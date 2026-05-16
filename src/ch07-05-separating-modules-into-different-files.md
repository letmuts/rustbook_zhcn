## 将模块分离到不同文件中（Separating Modules into Different Files）

到目前为止，本章中的所有示例都在一个文件中定义了多个模块。当模块变得很大时，你可能希望将其定义移到单独的文件中，以使代码更容易浏览。

例如，让我们从示例 7-17 中包含多个餐厅模块的代码开始。我们将模块提取到文件中，而不是将所有模块定义在 crate 根文件中。在本例中，crate 根文件是 _src/lib.rs_，但此过程也适用于 crate 根文件为 _src/main.rs_ 的二进制 crate。

首先，我们将 `front_of_house` 模块提取到自己的文件中。删除 `front_of_house` 模块大括号内的代码，只保留 `mod front_of_house;` 声明，这样 _src/lib.rs_ 包含示例 7-21 中所示的代码。请注意，在我们创建示例 7-22 中的 _src/front_of_house.rs_ 文件之前，这段代码不会编译。

<Listing number="7-21" file-name="src/lib.rs" caption="声明 `front_of_house` 模块，其函数体将在 *src/front_of_house.rs* 中">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

接下来，将原来在大括号内的代码放入一个名为 _src/front_of_house.rs_ 的新文件中，如示例 7-22 所示。编译器知道要在此文件中查找，因为它在 crate 根中遇到了名为 `front_of_house` 的模块声明。

<Listing number="7-22" file-name="src/front_of_house.rs" caption="*src/front_of_house.rs* 中 `front_of_house` 模块内部的定义">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

请注意，你只需要在模块树中**一次**使用 `mod` 声明来加载一个文件。一旦编译器知道该文件是项目的一部分（并且因为 `mod` 语句的位置而知道代码在模块树中的位置），项目中的其他文件应使用指向其声明位置的路径来引用该已加载文件的代码，如["引用模块树中项的路径"][paths]<!-- ignore -->一节所述。换句话说，`mod` **不是**你在其他编程语言中可能见过的"include"操作。

接下来，我们将 `hosting` 模块提取到自己的文件中。这个过程略有不同，因为 `hosting` 是 `front_of_house` 的子模块，而不是根模块的子模块。我们将 `hosting` 的文件放在一个以其在模块树中的祖先命名的新目录中，在本例中为 _src/front_of_house_。

要开始移动 `hosting`，我们将 _src/front_of_house.rs_ 更改为仅包含 `hosting` 模块的声明：

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

然后，我们创建一个 _src/front_of_house_ 目录和一个 _hosting.rs_ 文件，以包含 `hosting` 模块中的定义：

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

如果我们将 _hosting.rs_ 放在 _src_ 目录中，编译器会期望 _hosting.rs_ 中的代码位于 crate 根中声明的 `hosting` 模块中，而不是作为 `front_of_house` 模块的子模块声明。编译器用于检查哪些文件对应哪些模块代码的规则，意味着目录和文件与模块树更加匹配。

> ### 替代文件路径（Alternate File Paths）
>
> 到目前为止，我们已经介绍了 Rust 编译器使用的最惯用的文件路径，但 Rust 也支持一种较旧风格的文件路径。对于在 crate 根中声明的名为 `front_of_house` 的模块，编译器会在以下位置查找模块的代码：
>
> - _src/front_of_house.rs_（我们介绍的方式）
> - _src/front_of_house/mod.rs_（较旧的风格，仍支持）
>
> 对于作为 `front_of_house` 子模块的名为 `hosting` 的模块，编译器会在以下位置查找模块的代码：
>
> - _src/front_of_house/hosting.rs_（我们介绍的方式）
> - _src/front_of_house/hosting/mod.rs_（较旧的风格，仍支持）
>
> 如果你对同一个模块同时使用两种风格，将会收到编译器错误。在同一项目中对不同模块混合使用两种风格是允许的，但可能会让浏览你的项目的人感到困惑。
>
> 使用名为 _mod.rs_ 的文件风格的主要缺点是，你的项目可能最终会有许多名为 _mod.rs_ 的文件，当你在编辑器中同时打开它们时可能会造成混淆。

我们已将每个模块的代码移动到单独的文件中，而模块树保持不变。`eat_at_restaurant` 中的函数调用将无需任何修改即可正常工作，即使定义位于不同的文件中。这种技术让你可以在模块大小增长时将其移动到新文件中。

请注意，_src/lib.rs_ 中的 `pub use crate::front_of_house::hosting` 语句也没有改变，`use` 也对哪些文件被编译为 crate 的一部分没有任何影响。`mod` 关键字声明模块，Rust 在与模块同名的文件中查找该模块中的代码。

## 总结（Summary）

Rust 允许你将一个包（Package）拆分为多个 crate，并将一个 crate 拆分为多个模块（Module），以便你可以从一个模块引用另一个模块中定义的项（Item）。你可以通过指定绝对路径或相对路径来实现这一点。这些路径可以通过 `use` 语句引入作用域，这样你可以在该作用域中多次使用该项时使用更短的路径。模块代码默认是私有的，但你可以通过添加 `pub` 关键字使定义变为公共的。

在下一章中，我们将介绍标准库中的一些集合（Collection）数据结构，你可以在组织良好的代码中使用它们。

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
