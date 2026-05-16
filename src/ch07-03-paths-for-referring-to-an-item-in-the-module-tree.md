## 引用模块树中项的路径（Paths for Referring to an Item in the Module Tree）

为了告诉 Rust 在模块树中的何处找到某个项（Item），我们使用路径，就像在文件系统中导航时使用路径一样。要调用一个函数，我们需要知道它的路径。

路径有两种形式：

- **绝对路径**（Absolute Path）是从 crate 根开始的完整路径；对于来自外部 crate 的代码，绝对路径以 crate 名称开头，对于来自当前 crate 的代码，它以字面量 `crate` 开头。
- **相对路径**（Relative Path）从当前模块开始，使用 `self`、`super` 或当前模块中的标识符。

绝对路径和相对路径后都跟着一个或多个由双冒号（`::`）分隔的标识符。

回到示例 7-1，假设我们要调用 `add_to_waitlist` 函数。这等同于问：`add_to_waitlist` 函数的路径是什么？示例 7-3 包含了示例 7-1，但删除了一些模块和函数。

我们将展示两种方式，从 crate 根中定义的新函数 `eat_at_restaurant` 调用 `add_to_waitlist` 函数。这些路径是正确的，但仍然存在另一个问题，会阻止该示例按原样编译。我们稍后会解释原因。

`eat_at_restaurant` 函数是我们库 crate 公共 API（Public API）的一部分，因此我们使用 `pub` 关键字标记它。在[“使用 `pub` 关键字暴露路径”][pub]<!-- ignore -->一节中，我们将更详细地介绍 `pub`。

<Listing number="7-3" file-name="src/lib.rs" caption="使用绝对路径和相对路径调用 `add_to_waitlist` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

我们在 `eat_at_restaurant` 中第一次调用 `add_to_waitlist` 函数时，使用了绝对路径。`add_to_waitlist` 函数与 `eat_at_restaurant` 定义在同一个 crate 中，这意味着我们可以使用 `crate` 关键字来开始一个绝对路径。然后我们依次包含每个连续的模块，直到到达 `add_to_waitlist`。你可以想象一个具有相同结构的文件系统：我们会指定路径 `/front_of_house/hosting/add_to_waitlist` 来运行 `add_to_waitlist` 程序；使用 `crate` 名称从 crate 根开始，就像在 shell 中使用 `/` 从文件系统根开始一样。

我们在 `eat_at_restaurant` 中第二次调用 `add_to_waitlist` 时，使用了相对路径。路径以 `front_of_house` 开头，这是与 `eat_at_restaurant` 定义在同一模块树级别的模块名称。这里的文件系统等价物是使用路径 `front_of_house/hosting/add_to_waitlist`。以模块名开头意味着该路径是相对的。

选择使用相对路径还是绝对路径是一个基于项目做出的决定，取决于你是否更可能将项的定义代码与使用该项的代码分开移动还是一起移动。例如，如果我们将 `front_of_house` 模块和 `eat_at_restaurant` 函数移动到一个名为 `customer_experience` 的模块中，我们需要更新到 `add_to_waitlist` 的绝对路径，但相对路径仍然有效。但是，如果我们将 `eat_at_restaurant` 函数单独移动到一个名为 `dining` 的模块中，到 `add_to_waitlist` 调用的绝对路径将保持不变，但相对路径需要更新。我们通常倾向于指定绝对路径，因为我们更有可能希望彼此独立地移动代码定义和项调用。

让我们尝试编译示例 7-3，找出为什么它还不能编译！我们得到的错误如示例 7-4 所示。

<Listing number="7-4" caption="构建示例 7-3 中代码时产生的编译器错误">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

错误消息说 `hosting` 模块是私有的。换句话说，我们为 `hosting` 模块和 `add_to_waitlist` 函数使用了正确的路径，但 Rust 不允许我们使用它们，因为它无法访问私有部分。在 Rust 中，所有项（函数、方法、结构体、枚举、模块和常量）默认情况下对其父模块是私有的。如果你想让函数或结构体等项变为私有的，就把它放在一个模块中。

父模块中的项不能使用子模块中的私有项，但子模块中的项可以使用其祖先模块中的项。这是因为子模块包装并隐藏了它们的实现细节，但子模块可以看到它们定义所在的上下文。继续我们的比喻，可以将私有性规则想象成餐厅的后勤办公室：那里发生的事情对餐厅顾客来说是私有的，但办公室经理可以看到和做他们经营的餐厅中的一切。

Rust 选择让模块系统以这种方式运行，这样隐藏内部实现细节就是默认行为。这样，你就知道内部代码的哪部分可以在不破坏外部代码的情况下更改。但是，Rust 也赋予你使用 `pub` 关键字将子模块代码的内部部分暴露给外部祖先模块的能力。

### 使用 `pub` 关键字暴露路径（Exposing Paths with the `pub` Keyword）

让我们回到示例 7-4 中的错误，它告诉我们 `hosting` 模块是私有的。我们希望父模块中的 `eat_at_restaurant` 函数能够访问子模块中的 `add_to_waitlist` 函数，因此我们用 `pub` 关键字标记 `hosting` 模块，如示例 7-5 所示。

<Listing number="7-5" file-name="src/lib.rs" caption="将 `hosting` 模块声明为 `pub` 使其可从 `eat_at_restaurant` 使用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs}}
```

</Listing>

不幸的是，示例 7-5 中的代码仍然会导致错误，如示例 7-6 所示。

<Listing number="7-6" caption="构建示例 7-5 中代码时产生的编译器错误">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

发生了什么？在 `mod hosting` 前面添加 `pub` 关键字使模块变为公共的。通过这次更改，如果我们可以访问 `front_of_house`，我们就可以访问 `hosting`。但 `hosting` 的**内容**仍然是私有的；将模块设为公共的并不会使其内容变为公共的。模块上的 `pub` 关键字只允许其祖先模块中的代码引用它，而不能访问其内部代码。因为模块只是一些容器；仅仅将模块设为公共的，并不能做什么。你需要进一步选择将模块中的一个或多个项也设为公共的。

示例 7-6 中的错误说 `add_to_waitlist` 函数是私有的。私有性规则适用于结构体、枚举、函数和方法以及模块。

让我们也通过在 `add_to_waitlist` 函数的定义前添加 `pub` 关键字来使其变为公共的，如示例 7-7 所示。

<Listing number="7-7" file-name="src/lib.rs" caption="在 `mod hosting` 和 `fn add_to_waitlist` 前添加 `pub` 关键字让我们可以从 `eat_at_restaurant` 调用该函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

现在代码可以编译了！要了解为什么添加 `pub` 关键字让我们能在 `eat_at_restaurant` 中根据私有性规则使用这些路径，让我们看一下绝对路径和相对路径。

在绝对路径中，我们从 `crate` 开始，即 crate 模块树的根。`front_of_house` 模块定义在 crate 根中。虽然 `front_of_house` 不是公共的，但由于 `eat_at_restaurant` 函数与 `front_of_house` 定义在同一个模块中（即 `eat_at_restaurant` 和 `front_of_house` 是兄弟模块），我们可以从 `eat_at_restaurant` 引用 `front_of_house`。接下来是标记为 `pub` 的 `hosting` 模块。我们可以访问 `hosting` 的父模块，所以我们可以访问 `hosting`。最后，`add_to_waitlist` 函数标记为 `pub`，并且我们可以访问它的父模块，因此这个函数调用可以正常工作！

在相对路径中，逻辑与绝对路径相同，只是第一步不同：路径不是从 crate 根开始，而是从 `front_of_house` 开始。`front_of_house` 模块与 `eat_at_restaurant` 定义在同一个模块中，所以从定义 `eat_at_restaurant` 的模块开始的相对路径可以正常工作。然后，因为 `hosting` 和 `add_to_waitlist` 都标记为 `pub`，路径的其余部分也可以正常工作，这个函数调用是有效的！

如果你计划共享你的库 crate，以便其他项目可以使用你的代码，那么你的公共 API 就是你与 crate 用户之间的契约，决定了他们如何与你的代码交互。管理公共 API 更改有许多考虑因素，以使人们更容易依赖你的 crate。这些考虑因素超出了本书的范围；如果你对此主题感兴趣，请参阅 [Rust API 指南][api-guidelines]。

> #### 包含二进制和库的包的最佳实践（Best Practices for Packages with a Binary and a Library）
>
> 我们提到过，一个包可以同时包含一个 _src/main.rs_ 二进制 crate 根和一个 _src/lib.rs_ 库 crate 根，并且两个 crate 默认都具有包的名称。通常，这种同时包含库和二进制 crate 模式的包，其二进制 crate 中只包含足够的代码来启动一个可执行文件，该文件调用库 crate 中定义的代码。这让其他项目可以从该包提供的大部分功能中受益，因为库 crate 的代码可以被共享。
>
> 模块树应该定义在 _src/lib.rs_ 中。然后，任何公共项都可以通过以包名开头的路径在二进制 crate 中使用。二进制 crate 成为库 crate 的用户，就像完全外部的 crate 使用库 crate 一样：它只能使用公共 API。这有助于你设计一个好的 API；你不仅是作者，同时也是客户！
>
> 在[第 12 章][ch12]<!-- ignore -->中，我们将通过一个命令行程序来演示这种组织实践，该程序将同时包含二进制 crate 和库 crate。

### 使用 `super` 开始相对路径（Starting Relative Paths with `super`）

我们可以通过在路径开头使用 `super` 来构造从父模块开始的相对路径，而不是从当前模块或 crate 根开始。这就像用 `..` 语法开始文件系统路径，表示转到父目录。使用 `super` 允许我们引用我们知道在父模块中的项，当模块与父模块紧密相关但父模块将来可能被移动到模块树的其他位置时，这可以使重新排列模块树更容易。

考虑示例 7-8 中的代码，该代码模拟了厨师修正错误订单并亲自将其带给顾客的情况。定义在 `back_of_house` 模块中的 `fix_incorrect_order` 函数通过指定以 `super` 开头的路径来调用定义在父模块中的 `deliver_order` 函数。

<Listing number="7-8" file-name="src/lib.rs" caption="使用以 `super` 开头的相对路径调用函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` 函数在 `back_of_house` 模块中，因此我们可以使用 `super` 转到 `back_of_house` 的父模块，在本例中即 `crate`，即根模块。从那里，我们寻找 `deliver_order` 并找到了它。成功！我们认为 `back_of_house` 模块和 `deliver_order` 函数可能保持彼此相同的关系，并且在我们决定重新组织 crate 的模块树时可能会一起移动。因此，我们使用了 `super`，这样如果此代码被移动到不同的模块，将来需要更新的位置就更少。

### 使结构体和枚举变为公共的（Making Structs and Enums Public）

我们也可以使用 `pub` 将结构体和枚举指定为公共的，但在结构体和枚举上使用 `pub` 有一些额外的细节。如果我们在结构体定义前使用 `pub`，我们将结构体变为公共的，但结构体的字段仍然是私有的。我们可以根据具体情况决定每个字段是否公共。在示例 7-9 中，我们定义了一个公共的 `back_of_house::Breakfast` 结构体，其中包含一个公共的 `toast` 字段和一个私有的 `seasonal_fruit` 字段。这模拟了餐厅中顾客可以选择餐点附带的面包类型，但厨师根据当季和库存情况决定搭配餐点的水果的情况。可用的水果变化很快，因此顾客不能选择水果，甚至看不到他们会得到什么水果。

<Listing number="7-9" file-name="src/lib.rs" caption="一个包含一些公共字段和一些私有字段的结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

因为 `back_of_house::Breakfast` 结构体中的 `toast` 字段是公共的，所以在 `eat_at_restaurant` 中，我们可以使用点号表示法（Dot Notation）来读写 `toast` 字段。请注意，我们不能在 `eat_at_restaurant` 中使用 `seasonal_fruit` 字段，因为 `seasonal_fruit` 是私有的。尝试取消注释修改 `seasonal_fruit` 字段值的那行代码，看看你会得到什么错误！

另外，请注意，因为 `back_of_house::Breakfast` 有一个私有字段，该结构体需要提供一个公共关联函数（Associated Function）来构造 `Breakfast` 的实例（我们在这里将其命名为 `summer`）。如果 `Breakfast` 没有这样的函数，我们就无法在 `eat_at_restaurant` 中创建 `Breakfast` 的实例，因为我们无法在 `eat_at_restaurant` 中设置私有字段 `seasonal_fruit` 的值。

相比之下，如果我们将一个枚举变为公共的，那么它的所有变体会自动变为公共的。我们只需要在 `enum` 关键字之前加上 `pub`，如示例 7-10 所示。

<Listing number="7-10" file-name="src/lib.rs" caption="将一个枚举指定为公共的会使其所有变体都变为公共的">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

因为我们将 `Appetizer` 枚举设为公共的，所以我们可以在 `eat_at_restaurant` 中使用 `Soup` 和 `Salad` 变体。

除非枚举的变体是公共的，否则枚举并不是很有用；在每种情况下都必须为所有枚举变体添加 `pub` 注解是很烦人的，因此枚举变体默认是公共的。结构体在其字段不是公共的情况下通常也是有用的，因此结构体字段遵循默认情况下所有内容都是私有的通用规则，除非用 `pub` 注解。

还有一个涉及 `pub` 的情况我们尚未涵盖，那就是我们最后一个模块系统特性：`use` 关键字。我们将首先单独介绍 `use`，然后展示如何结合使用 `pub` 和 `use`。

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
