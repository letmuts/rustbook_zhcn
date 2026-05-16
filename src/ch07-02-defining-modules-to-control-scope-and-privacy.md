<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## 使用模块控制作用域和私有性（Control Scope and Privacy with Modules）

在本节中，我们将讨论模块（Module）和模块系统的其他部分，即**路径**（Path），它允许你命名项（Item）；`use` 关键字，将路径引入作用域；以及 `pub` 关键字，使项变为公共的。我们还将讨论 `as` 关键字、外部包（External Package）和全局运算符（Glob Operator）。

### 模块速查表（Modules Cheat Sheet）

在深入了解模块和路径的细节之前，我们在此提供一个快速参考，介绍模块、路径、`use` 关键字和 `pub` 关键字在编译器中的工作方式，以及大多数开发者如何组织他们的代码。我们将在本章中逐一举例说明每条规则，但这里是一个很好的参考位置，用于提醒模块的工作方式。

- **从 crate 根开始**：在编译一个 crate 时，编译器首先在 crate 根文件（对于库 crate 通常是 _src/lib.rs_，对于二进制 crate 通常是 _src/main.rs_）中寻找要编译的代码。
- **声明模块**：在 crate 根文件中，你可以声明新的模块；比如你用 `mod garden;` 声明一个 "garden" 模块。编译器会在以下位置寻找模块的代码：
  - 内联（Inline），在大括号内取代 `mod garden` 后面的分号
  - 在文件 _src/garden.rs_ 中
  - 在文件 _src/garden/mod.rs_ 中
- **声明子模块（Submodule）**：在除 crate 根之外的任何文件中，你都可以声明子模块。例如，你可以在 _src/garden.rs_ 中声明 `mod vegetables;`。编译器会在以父模块命名的目录中寻找子模块的代码，具体在以下位置：
  - 内联，直接跟在 `mod vegetables` 之后，在大括号内而不是分号
  - 在文件 _src/garden/vegetables.rs_ 中
  - 在文件 _src/garden/vegetables/mod.rs_ 中
- **模块中代码的路径**：一旦一个模块成为你的 crate 的一部分，只要私有性规则允许，你可以使用代码的路径从该 crate 的任何其他位置引用该模块中的代码。例如，garden vegetables 模块中的 `Asparagus` 类型将位于 `crate::garden::vegetables::Asparagus`。
- **私有 vs. 公共**：默认情况下，模块中的代码对其父模块是私有的。要使一个模块变为公共的，用 `pub mod` 而不是 `mod` 来声明它。要使公共模块中的项也变为公共的，在其声明前使用 `pub`。
- **`use` 关键字**：在一个作用域内，`use` 关键字创建项的快捷方式，以减少长路径的重复。在任何可以引用 `crate::garden::vegetables::Asparagus` 的作用域中，你可以用 `use crate::garden::vegetables::Asparagus;` 创建一个快捷方式，从此只需写 `Asparagus` 即可在该作用域中使用该类型。

在这里，我们创建一个名为 `backyard` 的二进制 crate 来说明这些规则。该 crate 的目录（同样命名为 _backyard_）包含以下文件和目录：

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

本例中的 crate 根文件是 _src/main.rs_，其内容如下：

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` 这一行告诉编译器包含在 _src/garden.rs_ 中找到的代码，该文件内容为：

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

在这里，`pub mod vegetables;` 表示 _src/garden/vegetables.rs_ 中的代码也被包含进来。该代码为：

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

现在让我们深入这些规则的细节并实际演示！

### 在模块中对相关代码进行分组（Grouping Related Code in Modules）

**模块**（Module）让我们在 crate 内组织代码，以提高可读性并便于重用。模块还允许我们控制项的**私有性**（Privacy），因为模块内的代码默认是私有的。私有项是内部实现细节，不可供外部使用。我们可以选择将模块及其内部的项设为公共的，从而将其暴露出来，允许外部代码使用和依赖它们。

举个例子，让我们编写一个提供餐厅功能的库 crate。我们将定义函数的签名（Signature），但函数体留空，以便专注于代码的组织而不是餐厅的实现。

在餐饮业中，餐厅的某些部分被称为前厅（Front of House），其他部分被称为后厨（Back of House）。**前厅**是顾客所在的地方；包括主人安排顾客就座、服务员接单和收款以及调酒师制作饮品的地方。**后厨**是厨师和烹饪人员在厨房工作、洗碗工清洁以及经理进行行政管理工作的地方。

为了以这种方式构建我们的 crate，我们可以将其函数组织到嵌套模块中。运行 `cargo new restaurant --lib` 创建一个名为 `restaurant` 的新库。然后将示例 7-1 中的代码输入到 _src/lib.rs_ 中，以定义一些模块和函数签名；这段代码是前厅部分。

<Listing number="7-1" file-name="src/lib.rs" caption="一个 `front_of_house` 模块，其中包含其他模块，这些模块又包含函数">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

我们使用 `mod` 关键字后跟模块名（在本例中为 `front_of_house`）来定义一个模块。模块体则放在大括号内。在模块内部，我们可以放置其他模块，就像本例中的 `hosting` 和 `serving` 模块。模块还可以容纳其他项的定义，例如结构体（Struct）、枚举（Enum）、常量（Constant）、特质（Trait），以及如示例 7-1 中的函数。

通过使用模块，我们可以将相关的定义组合在一起，并说明它们为何相关。使用此代码的程序员可以基于分组来浏览代码，而不必通读所有定义，从而更容易找到与他们相关的定义。向此代码添加新功能的程序员知道将代码放在哪里以保持程序的组织有序。

之前，我们提到 _src/main.rs_ 和 _src/lib.rs_ 被称为 **crate 根**（Crate Root）。这样命名的原因是，这两个文件中任何一个的内容会在 crate 模块结构的根部形成一个名为 `crate` 的模块，该结构被称为**模块树**（Module Tree）。

示例 7-2 展示了示例 7-1 中结构的模块树。

<Listing number="7-2" caption="示例 7-1 中代码的模块树">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

这棵树展示了某些模块如何嵌套在其他模块内部；例如，`hosting` 嵌套在 `front_of_house` 内部。该树还显示某些模块是**兄弟模块**（Sibling），这意味着它们定义在同一个模块中；`hosting` 和 `serving` 是定义在 `front_of_house` 内部的兄弟模块。如果模块 A 包含在模块 B 内部，我们说模块 A 是模块 B 的**子模块**（Child），而模块 B 是模块 A 的**父模块**（Parent）。请注意，整个模块树都根植于名为 `crate` 的隐式模块之下。

模块树可能会让你想起计算机上文件系统的目录树；这是一个非常贴切的类比！就像文件系统中的目录一样，你使用模块来组织代码。就像目录中的文件一样，我们需要一种方法来找到我们的模块。
