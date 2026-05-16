## 引用（References）与借用（Borrowing）

清单 4-5 中元组代码的问题在于，我们必须将 `String` 返回给调用函数，以便在调用 `calculate_length` 后仍能使用该 `String`，因为 `String` 被移动到了 `calculate_length` 中。相反，我们可以提供 `String` 值的一个引用（reference）。引用（reference）就像指针（pointer）一样，它是一个地址，我们可以通过它来访问存储在该地址的数据；这些数据由其他变量所拥有。与指针不同的是，引用保证在该引用的整个生命周期中都指向某个特定类型的有效值。

下面展示了如何定义并使用一个以对象引用作为参数（而非获取所有权）的 `calculate_length` 函数：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

首先，注意变量声明和函数返回值中所有的元组代码都消失了。其次，注意我们将 `&s1` 传递给 `calculate_length`，并且在函数定义中，我们使用 `&String` 而非 `String`。这些与符号（ampersands）代表引用（reference），它们允许你引用某个值而不获取其所有权。图 4-6 展示了这一概念。

<img alt="三张表格：s 的表格仅包含一个指向 s1 表格的指针。s1 的表格包含 s1 的栈数据，并指向堆上的字符串数据。" src="img/trpl04-06.svg" class="center" />

<span class="caption">图 4-6：`&String` `s` 指向 `String` `s1` 的示意图</span>

> 注意：与使用 `&` 进行引用（referencing）相反的操作是 _解引用（dereferencing）_，它通过解引用运算符 `*` 来完成。我们将在第 8 章看到解引用运算符的一些用法，并在第 15 章详细讨论解引用。

让我们更仔细地看一下这里的函数调用：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` 语法让我们创建一个 _引用（reference）_ `s1` 的值但不会拥有它。因为引用并不拥有该值，所以当引用停止使用时，它所指向的值不会被丢弃（dropped）。

同样地，函数的签名使用了 `&` 来表明参数 `s` 的类型是一个引用。让我们添加一些解释性的注释：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

变量 `s` 有效的作用域（scope）与任何函数参数的作用域相同，但当 `s` 停止使用时，引用所指向的值不会被丢弃，因为 `s` 没有所有权。当函数使用引用作为参数而不是实际值时，我们不需要返回值来交还所有权，因为我们从未拥有过所有权。

我们将创建引用的行为称为 _借用（borrowing）_。就像现实生活中，如果一个人拥有某样东西，你可以向他借用。当你用完时，你必须归还。你并不拥有它。

那么，如果我们试图修改正在借用的东西，会发生什么？试试清单 4-6 中的代码。剧透警告：它不会工作！

<Listing number="4-6" file-name="src/main.rs" caption="尝试修改借用的值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

正如变量默认是不可变的（immutable）一样，引用也是如此。我们不允许修改我们拥有引用的东西。

### 可变引用（Mutable References）

我们可以通过一些小的调整来修复清单 4-6 中的代码，使其允许我们修改一个借用的值，即使用 _可变引用（mutable reference）_：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

首先，我们将 `s` 改为 `mut`。然后，在调用 `change` 函数的地方使用 `&mut s` 创建一个可变引用，并更新函数签名以通过 `some_string: &mut String` 接受一个可变引用。这清楚地表明 `change` 函数将会改变它借用的值。

可变引用有一个很大的限制：如果你有一个对某个值的可变引用，你就不能再有对该值的其他引用。这段试图创建两个对 `s` 的可变引用的代码将会失败：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

这个错误说明此代码无效，因为我们不能在同一时间多次将 `s` 作为可变借用。第一个可变借用是在 `r1` 中，并且必须持续到它在 `println!` 中被使用，但在该可变引用的创建与其使用之间，我们试图在 `r2` 中创建另一个可变引用，该引用借用了与 `r1` 相同的数据。

这个限制——禁止同一时间对同一数据存在多个可变引用——允许进行修改，但以一种非常受控的方式进行。这是新的 Rustacean（Rust 学习者）常常感到困扰的地方，因为大多数语言允许你随时随地进行修改。拥有这个限制的好处是 Rust 能够在编译时防止数据竞争（data race）。_数据竞争（data race）_ 类似于竞态条件（race condition），当以下三种行为同时发生时就会发生：

- 两个或更多指针同时访问同一数据。
- 至少有一个指针被用来写入数据。
- 没有使用任何机制来同步对数据的访问。

数据竞争会导致未定义行为（undefined behavior），并且在运行时追踪排查时很难诊断和修复；Rust 通过拒绝编译存在数据竞争的代码来防止这个问题！

和往常一样，我们可以使用花括号创建一个新的作用域（scope），允许有多个可变引用，只是不能 _同时（simultaneous）_ 存在：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust 对组合可变引用和不可变引用也实施了类似的规则。以下代码会导致错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

呼！我们在拥有对同一个值的不可变引用时，_也_ 不能拥有可变引用。

不可变引用的使用者并不期望值突然发生变化！然而，多个不可变引用是允许的，因为仅仅读取数据的人没有能力影响其他人对数据的读取。

请注意，引用的作用域从它被引入的地方开始，一直持续到该引用的最后一次使用。例如，以下代码可以编译，因为不可变引用的最后一次使用是在 `println!` 中，这发生在可变引用被引入之前：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

不可变引用 `r1` 和 `r2` 的作用域在它们最后被使用的 `println!` 之后结束，这发生在可变引用 `r3` 创建之前。这些作用域不重叠，因此这段代码是允许的：编译器可以在作用域结束之前的某个时间点判断出该引用已不再使用。

尽管借用错误有时可能会令人沮丧，但请记住，这是 Rust 编译器在早期（编译时而非运行时）指出潜在的 bug，并精确地告诉你问题所在。这样，你就不必在之后追踪为什么数据不是你所想的那样了。

### 悬垂引用（Dangling References）

在存在指针的语言中，很容易错误地创建 _悬垂指针（dangling pointer）_——即指向某个内存位置的指针，该内存可能已被分配给其他人——通过释放某些内存的同时却保留指向该内存的指针。相比之下，在 Rust 中，编译器保证引用永远不会是悬垂引用：如果你拥有对某些数据的引用，编译器将确保数据不会在引用之前超出作用域。

让我们尝试创建一个悬垂引用，看看 Rust 如何通过编译时错误来防止它们：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

以下是错误信息：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

这个错误信息提到了一个我们尚未涉及的特性：生命周期（lifetimes）。我们将在第 10 章详细讨论生命周期。但是，如果你忽略关于生命周期的部分，该信息确实包含了为什么这段代码有问题的关键：

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

让我们更仔细地看看 `dangle` 代码的每个阶段具体发生了什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

因为 `s` 是在 `dangle` 内部创建的，当 `dangle` 的代码执行完毕后，`s` 将被释放。但我们试图返回一个对它的引用。这意味着这个引用将指向一个无效的 `String`。这可不行！Rust 不允许我们这样做。

这里的解决方案是直接返回 `String`：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

这样就能正常工作，没有任何问题。所有权被移出，没有任何东西被释放。

### 引用的规则

让我们回顾一下关于引用所讨论的内容：

- 在任意给定时间，你 _要么_ 只能有一个可变引用，_要么_ 只能有任意数量的不可变引用。
- 引用必须始终有效。

接下来，我们将介绍另一种引用：切片（slices）。
