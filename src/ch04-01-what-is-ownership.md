## 什么是所有权？（What Is Ownership?）

**所有权（Ownership）** 是一组规则，用于管理 Rust 程序如何管理内存。
所有程序在运行时都必须管理它们使用计算机内存的方式。
有些语言具有垃圾回收机制（garbage collection），可以在程序运行时定期查找不再使用的内存；
而在其他语言中，程序员必须显式地分配（allocate）和释放（free）内存。
Rust 采用了第三种方式：通过一套由编译器检查的所有权规则来管理内存。
如果违反了任何规则，程序将无法编译。
所有权的任何特性都不会在运行时降低程序的速度。

因为所有权对许多程序员来说是一个新概念，确实需要一些时间来适应。
好消息是，随着你对 Rust 和所有权系统规则的了解越来越深入，
你就会越自然地编写出既安全又高效的代码。坚持下去！

当你理解了所有权，你就为理解 Rust 的独特特性奠定了坚实的基础。
在本章中，你将通过一些专注于一种非常常见的数据结构——字符串——的示例来学习所有权。

> ### 栈（Stack）与堆（Heap）
>
> 许多编程语言不要求你经常考虑栈和堆的问题。
> 但在像 Rust 这样的系统编程语言中，一个值位于栈上还是堆上会影响语言的行为方式以及你为什么必须做出某些决定。
> 所有权的部分内容将在本章后面结合栈和堆进行描述，因此这里先做一个简单的解释作为准备。
>
> 栈和堆都是运行时可供代码使用的内存部分，但它们的结构方式不同。
> 栈按照获取值的顺序存储值，并以相反的顺序移除值。这被称为**后进先出（last in, first out，LIFO）**。
> 想象一摞盘子：当你添加更多盘子时，你将其放在堆叠的顶部，当你需要盘子时，你从顶部取下一个。
> 从中间或底部添加或移除盘子就不太行了！添加数据被称为**入栈（pushing onto the stack）**，
> 移除数据被称为**出栈（popping off the stack）**。
> 所有存储在栈上的数据必须具有已知的、固定的大小。
> 在编译时大小未知或大小可能变化的数据必须存储在堆上。
>
> 堆的组织性较差：当你将数据放在堆上时，你请求一定数量的空间。
> 内存分配器（memory allocator）会在堆中找到一个足够大的空位，将其标记为正在使用，
> 并返回一个**指针（pointer）**，即该位置的地址。
> 这个过程被称为**在堆上分配（allocating on the heap）**，有时简称为**分配（allocating）**
> （将值推入栈不被视为分配）。
> 因为指向堆的指针是已知的、固定大小的，你可以将指针存储在栈上，但当你需要实际数据时，你必须跟随指针。
> 想象在餐厅就座的情景。当你进入时，你说出同行人数，服务员找到一张适合所有人的空桌子并带你过去。
> 如果你的团队中有人迟到，他们可以询问你坐在哪里来找到你。
>
> 入栈比在堆上分配更快，因为分配器不需要搜索存储新数据的位置；该位置始终在栈顶。
> 相比之下，在堆上分配空间需要更多工作，因为分配器必须首先找到足够大的空间来存放数据，
> 然后执行簿记（bookkeeping）工作以为下一次分配做准备。
>
> 访问堆上的数据通常比访问栈上的数据慢，因为你需要跟随指针才能到达那里。
> 当代处理器在内存中跳跃较少时速度更快。继续这个类比，考虑一个餐厅服务员为多张桌子点单。
> 最有效的方式是在转到下一张桌子之前，先把一张桌子的所有订单都拿到。
> 从 A 桌点单，然后 B 桌点单，再回到 A 桌，再到 B 桌，这会慢得多。
> 同理，处理器在处理与其他数据接近的数据（如在栈上）时，通常比处理更远的数据（如在堆上）时表现得更好。
>
> 当你的代码调用函数时，传递给函数的值（可能包括指向堆上数据的指针）以及函数的局部变量会被推入栈中。
> 当函数结束时，这些值会被弹出栈。
>
> 跟踪哪些代码正在使用堆上的哪些数据，最小化堆上的重复数据量，
> 以及清理堆上未使用的数据以避免空间耗尽，这些都是所有权（ownership）所解决的问题。
> 一旦你理解了所有权，你就不需要经常考虑栈和堆了。
> 但知道所有权的主要目的是管理堆数据，可以帮助解释它为什么以这种方式工作。

### 所有权规则（Ownership Rules）

首先，让我们来看看所有权的规则。在我们学习说明这些规则的示例时，请牢记这些规则：

- Rust 中的每个值都有一个**所有者（owner）**。
- 同一时间只能有一个所有者。
- 当所有者离开作用域（scope）时，该值将被丢弃（dropped）。

### 变量作用域（Variable Scope）

既然我们已经过了基础的 Rust 语法阶段，就不会在示例中包含所有 `fn main() {`
代码，所以如果你在跟着练习，请确保手动将以下示例放入 `main` 函数中。
这样一来，我们的示例会更加简洁，让我们能够专注于实际细节而不是样板代码。

作为所有权的第一个示例，我们来看看一些变量的作用域。
**作用域（scope）** 是程序中一个项（item）有效的范围。以下面的变量为例：

```rust
let s = "hello";
```

变量 `s` 引用了一个字符串字面量（string literal），其中字符串的值被硬编码在程序的文本中。
该变量从声明之处开始直到当前作用域结束都是有效的。示例 4-1 展示了一个程序及其注释，
标明了变量 `s` 在何处是有效的。

<Listing number="4-1" caption="一个变量及其有效的作用域（A variable and the scope in which it is valid）">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

换句话说，这里有两个重要的时间点：

- 当 `s` **进入**作用域时，它是有效的。
- 它一直保持有效，直到它**离开**作用域。

至此，作用域与变量有效时间之间的关系与其他编程语言类似。
现在，我们将在此基础上通过引入 `String` 类型来进行进一步的学习。

### `String` 类型（The `String` Type）

为了说明所有权（ownership）的规则，我们需要一个比第 3 章的[“数据类型”][data-types]<!-- ignore -->部分所介绍的更为复杂的数据类型。
之前介绍的类型都是已知大小的，可以存储在栈上，并在其作用域结束时从栈中弹出，
并且如果需要代码的另一部分在不同的作用域中使用相同的值，可以快速、简单地复制以创建一个新的独立实例。
但我们想要看看存储在堆上的数据，并探索 Rust 如何知道何时清理这些数据，
而 `String` 类型正是一个很好的例子。

我们将重点讨论 `String` 中与所有权相关的部分。这些方面也适用于其他复杂的数据类型，
无论它们是标准库提供的还是你自己创建的。我们将在[第 8 章][ch8]<!-- ignore -->中讨论 `String` 中与所有权无关的方面。

我们已经见过字符串字面量（string literal），即字符串值被硬编码（hardcoded）到程序中。
字符串字面量很方便，但它们并不适用于所有我们可能想要使用文本的场景。
一个原因是它们是不可变的（immutable）。另一个原因是并非每个字符串值在编写代码时就能确定：
例如，如果我们想要获取用户输入并存储它呢？正是为了应对这些情况，Rust 提供了 `String` 类型。
该类型管理在堆上分配的数据，因此能够存储在编译时未知大小的文本量。
你可以使用 `from` 函数从字符串字面量创建一个 `String`，如下所示：

```rust
let s = String::from("hello");
```

双冒号 `::` 运算符允许我们将这个特定的 `from` 函数放在 `String` 类型的命名空间下，
而不是使用像 `string_from` 这样的名称。我们将在第 5 章的[“方法”][methods]<!-- ignore -->部分更详细地讨论这种语法，
并在第 7 章中讨论[“用于在模块树中引用项的路径”][paths-module-tree]<!-- ignore -->时讲解模块的命名空间。

这种字符串**是**可以修改的：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

那么，这里的区别是什么呢？为什么 `String` 可以修改而字面量却不能？
区别在于这两种类型处理内存的方式不同。

### 内存与分配（Memory and Allocation）

对于字符串字面量（string literal），我们在编译时就知道其内容，因此文本被直接硬编码（hardcoded）到最终的可执行文件中。
这就是字符串字面量快速且高效的原因。但这些特性仅仅源于字符串字面量的不可变性（immutability）。
不幸的是，对于每一段在编译时大小未知且可能在程序运行时改变大小的文本，
我们不能将一块内存放入二进制文件中。

对于 `String` 类型，为了支持可变（mutable）、可增长的文本片段，
我们需要在堆上分配一块在编译时大小未知的内存来存放内容。这意味着：

- 必须在运行时向内存分配器（memory allocator）请求内存。
- 当我们使用完 `String` 后，需要一种方式将这块内存返还给分配器。

第一部分由我们完成：当我们调用 `String::from` 时，其实现会请求它所需的内存。
这在编程语言中几乎是通用的。

然而，第二部分则有所不同。在拥有**垃圾回收器（garbage collector，GC）**的语言中，
GC 会跟踪并清理不再使用的内存，我们无需考虑这个问题。
在没有 GC 的大多数语言中，我们有责任识别内存何时不再被使用并显式调用代码来释放它，
就像我们请求它一样。历史上，正确做到这一点一直是一个困难的编程问题。
如果我们忘记了，就会浪费内存。如果我们释放得太早，就会产生一个无效的变量。
如果我们释放两次，那也是一个错误。我们需要恰好配对一次 `allocate` 与一次 `free`。

Rust 采取了一条不同的路径：一旦拥有它的变量离开作用域（scope），内存就会被自动返回。
下面是我们使用 `String` 而非字符串字面量的作用域示例（来自示例 4-1）的一个版本：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

有一个自然的时机可以将我们的 `String` 所需的内存返还给分配器：当 `s` 离开作用域时。
当变量离开作用域，Rust 会为我们调用一个特殊的函数。这个函数被称为 `drop`，
`String` 的作者可以在其中放置返还内存的代码。
Rust 会在右花括号处自动调用 `drop`。

> 注意：在 C++ 中，这种在项（item）生命周期（lifetime）结束时释放资源的模式有时被称为
> **资源获取即初始化（Resource Acquisition Is Initialization，RAII）**。
> 如果你使用过 RAII 模式，那么 Rust 中的 `drop` 函数会让你感到很熟悉。

这种模式对 Rust 代码的编写方式有着深远的影响。目前看来可能很简单，
但在更复杂的情况下，当我们希望有多个变量使用我们在堆上分配的数据时，
代码的行为可能会出乎意料。现在让我们探讨其中一些情况。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="ways-variables-and-data-interact-move"></a>

#### 变量与数据的交互方式：移动（Variables and Data Interacting with Move）

多个变量可以在 Rust 中以不同的方式与相同的数据进行交互。
示例 4-2 展示了一个使用整数的示例。

<Listing number="4-2" caption="将变量 `x` 的整数值赋值给 `y`（Assigning the integer value of variable `x` to `y`）">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

我们大概可以猜出这段代码的作用："将值 `5` 绑定到 `x`；然后，复制 `x` 中的值并将其绑定到 `y`。"
现在我们有两个变量 `x` 和 `y`，且都等于 `5`。这确实就是发生的事情，
因为整数是具有已知、固定大小的简单值，这两个 `5` 值被推入栈中。

现在让我们看看 `String` 版本：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

这看起来非常相似，因此我们可能认为它的工作方式也是相同的：
也就是说，第二行会复制 `s1` 中的值并将其绑定到 `s2`。
但实际上并非如此。

看一下图 4-1，了解 `String` 在底层的实际运作方式。
一个 `String` 由三部分组成，如左侧所示：一个指向存放字符串内容的内存的指针（pointer）、一个长度（length）和一个容量（capacity）。
这组数据存储在栈上。右侧是堆上存放内容的内存。

<img alt="两个表格：第一个表格包含 s1 在栈上的表示，由其长度（5）、容量（5）以及指向第二个表格中第一个值的指针组成。第二个表格包含堆上字符串数据的逐字节表示。" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-1：一个持有值 `"hello"` 且绑定到 `s1` 的 `String` 在内存中的表示（The representation in memory of a `String` holding the value `"hello"` bound to `s1`）</span>

长度（length）表示 `String` 的内容当前使用的内存字节数。
容量（capacity）表示 `String` 从分配器获得的内存总量。
长度和容量的差异在某些情况下很重要，但在此上下文中并不关键，因此目前可以忽略容量。

当我们将 `s1` 赋值给 `s2` 时，会复制 `String` 的数据，即复制栈上的指针、长度和容量。
我们不会复制指针所指向的堆上的数据。换句话说，内存中的数据表示如图 4-2 所示。

<img alt="三个表格：表格 s1 和 s2 分别表示栈上的这些字符串，并且都指向堆上相同的字符串数据。" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-2：变量 `s2` 拥有 `s1` 的指针、长度和容量副本的内存表示（The representation in memory of the variable `s2` that has a copy of the pointer, length, and capacity of `s1`）</span>

该表示**并非**如图 4-3 所示，即如果 Rust 也复制了堆数据时的内存情况。
如果 Rust 这样做，那么当堆上的数据很大时，操作 `s2 = s1` 在运行时性能方面可能会非常昂贵。

<img alt="四个表格：两个表格分别表示 s1 和 s2 的栈数据，每个表格指向堆上各自独立的字符串数据副本。" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-3：如果 Rust 也复制了堆数据时 `s2 = s1` 可能采取的另一种情况（Another possibility for what `s2 = s1` might do if Rust copied the heap data as well）</span>

之前我们说过，当变量离开作用域时，Rust 会自动调用 `drop` 函数并清理该变量的堆内存。
但图 4-2 显示两个数据指针指向同一个位置。这就产生了一个问题：
当 `s2` 和 `s1` 离开作用域时，它们都会尝试释放同一块内存。
这就是所谓的**二次释放（double free）**错误，也是我们之前提到的内存安全 bug 之一。
两次释放内存可能导致内存损坏（memory corruption），进而可能引发安全漏洞。

为了确保内存安全，在 `let s2 = s1;` 这一行之后，Rust 认为 `s1` 不再有效。
因此，当 `s1` 离开作用域时，Rust 不需要释放任何东西。
看看当你尝试在创建 `s2` 后使用 `s1` 会发生什么；它将无法工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

你会得到这样的错误，因为 Rust 阻止你使用失效的引用：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

如果你在其他语言中听说过**浅拷贝（shallow copy）**和**深拷贝（deep copy）**这两个术语，
那么复制指针、长度和容量而不复制数据的概念听起来可能像浅拷贝。
但由于 Rust 还会使第一个变量失效，这就不叫浅拷贝，而是被称为**移动（move）**。
在这个例子中，我们会说 `s1` 被**移动（moved）**到了 `s2` 中。因此，实际发生的情况如图 4-4 所示。

<img alt="三个表格：表格 s1 和 s2 分别表示栈上的这些字符串，并且都指向堆上相同的字符串数据。表格 s1 变为灰色，因为 s1 不再有效；只有 s2 可用于访问堆数据。" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-4：`s1` 被失效后的内存表示（The representation in memory after `s1` has been invalidated）</span>

这就解决了我们的问题！只有 `s2` 有效，当它离开作用域时，它将独自释放内存，一切就完成了。

此外，这还隐含着一个设计选择：Rust 永远不会自动创建数据的"深"拷贝。
因此，任何**自动**拷贝都可以被假定为在运行时性能方面是廉价的。

#### 作用域与赋值（Scope and Assignment）

反过来，作用域、所有权以及通过 `drop` 函数释放内存之间的关系也同样成立。
当你为一个现有变量赋一个全新的值时，Rust 会调用 `drop` 并立即释放原值的内存。
例如，考虑下面的代码：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

我们首先声明了一个变量 `s` 并将其绑定到一个值为 `"hello"` 的 `String`。
然后，我们立即创建了一个值为 `"ahoy"` 的新 `String` 并将其赋值给 `s`。
此时，没有任何内容再指向堆上的原值。图 4-5 展示了现在的栈和堆数据：

<img alt="一个表格表示栈上的字符串值，指向堆上的第二段字符串数据（ahoy），原始字符串数据（hello）变为灰色，因为它无法再被访问。" src="img/trpl04-05.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-5：初始值被完全替换后的内存表示（The representation in memory after the initial value has been replaced in its entirety）</span>

因此，原始字符串立即离开作用域。Rust 会对其调用 `drop` 函数，其内存也会被立即释放。
当我们在最后打印该值时，它将是 `"ahoy, world!"`。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="ways-variables-and-data-interact-clone"></a>

#### 变量与数据的交互方式：克隆（Variables and Data Interacting with Clone）

如果我们**确实**想要深度复制 `String` 的堆数据，而不仅仅是栈数据，
我们可以使用一个名为 `clone` 的通用方法。我们将在第 5 章讨论方法语法，
但由于方法是许多编程语言中的常见特性，你可能之前已经见过它们。

下面是 `clone` 方法的一个示例：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

这完全可行，并且明确地产生了图 4-3 所示的行为，其中堆数据**确实**被复制了。

当你看到对 `clone` 的调用时，你就知道正在执行一些任意代码，并且这些代码可能是昂贵的。
它是一个视觉指示器，表明有不同的事情正在发生。

#### 仅栈数据：复制（Stack-Only Data: Copy）

还有一个我们尚未讨论的细节。下面这段使用整数的代码（其中的一部分已在示例 4-2 中展示）是有效的：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

但这段代码似乎与我们刚刚学到的相矛盾：我们没有调用 `clone`，
但 `x` 仍然有效且并未被移动（moved）到 `y` 中。

原因是像整数这样在编译时具有已知大小的类型完全存储在栈上，
因此复制实际的值非常快速。这意味着我们没有理由要在创建变量 `y` 后阻止 `x` 仍然有效。
换句话说，在这里深拷贝和浅拷贝没有区别，
因此调用 `clone` 与通常的浅拷贝没有什么不同，我们可以省略它。

Rust 有一个特殊的标注（annotation）叫做 `Copy` trait（特征），
我们可以将其放在存储在栈上的类型上，就像整数一样（我们将在[第 10 章][traits]<!-- ignore -->中更详细地讨论 trait）。
如果一个类型实现了 `Copy` trait，那么使用它的变量不会被移动，而是会被简单地复制，
这使得它们在赋值给另一个变量后仍然有效。

如果一个类型或其任何部分实现了 `Drop` trait，Rust 不会允许我们对该类型标注 `Copy`。
如果该类型在值离开作用域时需要做一些特殊的事情，而我们在该类型上添加了 `Copy` 标注，
就会得到一个编译时错误。要了解如何为你的类型添加 `Copy` 标注以实现该 trait，
请参阅附录 C 中的[“可派生 trait”][derivable-traits]<!-- ignore -->。

那么，哪些类型实现了 `Copy` trait？你可以查看给定类型的文档来确认，
但作为一般规则，任何一组简单的标量值都可以实现 `Copy`，
而任何需要分配或某种形式的资源则不能实现 `Copy`。以下是一些实现了 `Copy` 的类型：

- 所有整数类型，例如 `u32`。
- 布尔类型 `bool`，其值为 `true` 和 `false`。
- 所有浮点数类型，例如 `f64`。
- 字符类型 `char`。
- 元组（tuple），前提是它们只包含也实现了 `Copy` 的类型。例如，
  `(i32, i32)` 实现了 `Copy`，但 `(i32, String)` 则没有。

### 所有权与函数（Ownership and Functions）

将值传递给函数的机制与将值赋值给变量类似。
将变量传递给函数会进行移动（move）或复制（copy），就像赋值一样。
示例 4-3 是一个带有标注的示例，显示变量在何处进入和离开作用域。

<Listing number="4-3" file-name="src/main.rs" caption="带有所有权和作用域标注的函数（Functions with ownership and scope annotated）">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

如果在调用 `takes_ownership` 之后尝试使用 `s`，Rust 会抛出一个编译时错误。
这些静态检查保护我们免于犯错。尝试向 `main` 中添加使用 `s` 和 `x` 的代码，
看看在何处可以使用它们，以及在何处所有权规则会阻止你这样做。

### 返回值与作用域（Return Values and Scope）

返回值也可以转移所有权。示例 4-4 展示了一个返回某个值的函数示例，
并带有与示例 4-3 类似的标注。

<Listing number="4-4" file-name="src/main.rs" caption="转移返回值的所有权（Transferring ownership of return values）">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

变量的所有权每次都遵循相同的模式：将一个值赋值给另一个变量会移动它。
当包含堆上数据的变量离开作用域时，该值将被 `drop` 清理，除非数据的所有权已经移动到另一个变量。

虽然这样可行，但在每个函数中获取所有权然后再返回所有权有点繁琐。
如果我们想让一个函数使用某个值但不获取所有权呢？
如果我们想再次使用传入的值，除了函数体中可能想要返回的任何数据之外，
还需要将其传回来，这相当烦人。

Rust 确实允许我们使用元组（tuple）返回多个值，如示例 4-5 所示。

<Listing number="4-5" file-name="src/main.rs" caption="返回参数的所有权（Returning ownership of parameters）">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

但这对于一个本应常见的概念来说，仪式感太强，工作量也太大了。
幸运的是，Rust 有一个功能可以在不转移所有权的情况下使用值：引用（references）。


[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[methods]: ch05-03-method-syntax.html#methods
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
