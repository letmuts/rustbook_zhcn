## 注释（Comments）

所有程序员都努力使他们的代码易于理解，但有时额外的解释也是必要的。在这种情况下，程序员会在源代码中留下 _注释（comments）_，编译器会忽略它们，但阅读源代码的人可能会觉得它们很有用。

这是一个简单的注释：

```rust
// hello, world
```

在 Rust 中，惯用的注释风格是用两个斜杠开始一个注释，并且注释一直延续到行尾。对于超过一行的注释，你需要在每一行都包含 `//`，就像这样：

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

注释也可以放在包含代码的行的末尾：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

但更常见的是以下面这种格式使用注释，将注释放在它所注解的代码上方的单独一行：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust 还有另一种注释，即文档注释（documentation comments），我们将在第 14 章的[“将 Crate 发布到 Crates.io”][publishing]<!-- ignore --> 一节中讨论。

[publishing]: ch14-02-publishing-to-crates-io.html
