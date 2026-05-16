# 理解所有权（Understanding Ownership）

所有权（Ownership）是 Rust 最独特的特性，并对语言的其余部分有着深远的影响。它使 Rust 能够在不需要垃圾收集器（garbage collector）的情况下保证内存安全（memory safety），因此理解所有权的工作原理非常重要。在本章中，我们将讨论所有权以及几个相关的特性：借用（borrowing）、切片（slices）以及 Rust 如何在内存中布局数据。
