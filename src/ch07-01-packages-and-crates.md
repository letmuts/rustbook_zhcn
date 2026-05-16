## 包（Package）和 Crate（Packages and Crates）

我们将介绍的模块系统的第一部分是包（Package）和 crate。

**Crate** 是 Rust 编译器在单次处理中考虑的最小代码量。即使你运行 `rustc` 而不是 `cargo`，并传递单个源代码文件（正如我们在第 1 章的[“Rust 程序基础”][basics]<!-- ignore -->中所做的那样），编译器也会将该文件视为一个 crate。Crate 可以包含模块（Module），模块可以定义在其他文件中，并与 crate 一起编译，我们将在接下来的小节中看到。

Crate 有两种形式：二进制 crate（Binary Crate）或库 crate（Library Crate）。**二进制 crate** 是可以编译为可执行文件的程序，比如命令行程序或服务器。每个二进制 crate 必须有一个名为 `main` 的函数，定义可执行文件运行时发生什么。到目前为止，我们创建的所有 crate 都是二进制 crate。

**库 crate** 没有 `main` 函数，也不会编译为可执行文件。相反，它们定义了旨在与多个项目共享的功能。例如，我们在[第 2 章][rand]<!-- ignore -->中使用的 `rand` crate 提供了生成随机数的功能。大多数情况下，当 Rust 程序员说"crate"时，他们指的是库 crate，并且他们将"crate"与通用编程概念中的"库"（Library）混用。

**Crate 根**（Crate Root）是一个源代码文件，Rust 编译器从它开始，它构成了你的 crate 的根模块（Root Module）（我们将在[“使用模块控制作用域和私有性”][modules]<!-- ignore -->中深入讲解模块）。

**包**（Package）是一个或多个 crate 的集合，提供一组功能。一个包包含一个 _Cargo.toml_ 文件，描述如何构建这些 crate。Cargo 本身实际上就是一个包，它包含了我们一直用来构建代码的命令行工具的二进制 crate。Cargo 包还包含一个该二进制 crate 所依赖的库 crate。其他项目可以依赖 Cargo 库 crate，来使用 Cargo 命令行工具所使用的相同逻辑。

一个包可以包含任意多的二进制 crate，但最多只能包含一个库 crate。一个包必须至少包含一个 crate，无论是库 crate 还是二进制 crate。

让我们看看创建包时会发生什么。首先，我们输入命令 `cargo new my-project`：

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

运行 `cargo new my-project` 后，我们使用 `ls` 来查看 Cargo 创建了什么。在 _my-project_ 目录中，有一个 _Cargo.toml_ 文件，这就是一个包。还有一个 _src_ 目录，其中包含 _main.rs_。在文本编辑器中打开 _Cargo.toml_，注意其中没有提到 _src/main.rs_。Cargo 遵循一个约定，即 _src/main.rs_ 是与包同名的二进制 crate 的 crate 根。同样，Cargo 知道如果包目录中包含 _src/lib.rs_，则该包包含一个与包同名的库 crate，并且 _src/lib.rs_ 是其 crate 根。Cargo 将 crate 根文件传递给 `rustc` 来构建库或二进制文件。

在这里，我们有一个只包含 _src/main.rs_ 的包，这意味着它只包含一个名为 `my-project` 的二进制 crate。如果一个包同时包含 _src/main.rs_ 和 _src/lib.rs_，则它有两个 crate：一个二进制 crate 和一个库 crate，两者都与包同名。一个包可以通过在 _src/bin_ 目录中放置文件来拥有多个二进制 crate：每个文件都将是一个单独的二进制 crate。

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
