## 切片（Slice）类型

*切片（Slice）* 让你可以引用[集合](ch08-00-common-collections.md)<!-- ignore -->中一段连续的元素序列。切片（Slice）是一种引用，因此它没有所有权。

这里有一个小编程问题：编写一个函数，它接收一个用空格分隔单词的字符串，并返回该字符串中找到的第一个单词。如果函数在字符串中没有找到空格，则整个字符串就是一个单词，所以应该返回整个字符串。

> 注意：为了介绍切片（Slice），本节仅假设使用 ASCII 字符；关于 UTF-8 处理的更详细讨论在第 8 章的["使用字符串存储 UTF-8 编码的文本"][strings]<!-- ignore -->部分。

让我们先来思考一下，如果不使用切片（Slice），我们该如何编写这个函数的签名，以便理解切片（Slice）要解决的问题：

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` 函数有一个 `&String` 类型的参数。我们不需要所有权，所以这没问题。（在惯用的 Rust 代码中，函数除非必要，否则不会获取参数的所有权，原因我们将在后续中逐渐明了。）但是我们应该返回什么呢？我们确实没有办法来描述字符串的*一部分*。不过，我们可以返回单词结尾的索引（Index），由空格来指示。让我们试试看，如示例 4-7 所示。

<Listing number="4-7" file-name="src/main.rs" caption="`first_word` 函数返回 `String` 参数中的一个字节索引值">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

因为我们需要逐元素地检查 `String` 中的值是否为空格，所以我们将使用 `as_bytes` 方法将 `String` 转换为字节数组。

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

接下来，我们使用 `iter` 方法在字节数组上创建一个迭代器（Iterator）：

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

我们将在[第 13 章][ch13]<!-- ignore -->中更详细地讨论迭代器（Iterator）。现在，你只需要知道 `iter` 是一个返回集合中每个元素的方法，而 `enumerate` 包装了 `iter` 的结果，将每个元素作为元组（Tuple）的一部分返回。`enumerate` 返回的元组中的第一个元素是索引（Index），第二个元素是对该元素的引用。这比我们自己计算索引要方便一些。

因为 `enumerate` 方法返回一个元组（Tuple），我们可以使用模式（Pattern）来解构这个元组。我们将在[第 6 章][ch6]<!-- ignore -->中更多地讨论模式（Pattern）。在 `for` 循环中，我们指定了一个模式：元组中的索引使用 `i`，元组中的单个字节使用 `&item`。因为我们从 `.iter().enumerate()` 获取的是元素的引用，所以我们在模式中使用了 `&`。

在 `for` 循环内部，我们使用字节字面量语法来搜索代表空格的字节。如果找到空格，就返回该位置。否则，使用 `s.len()` 返回字符串的长度。

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

现在我们有办法找出字符串中第一个单词结尾的索引（Index），但有一个问题。我们返回了一个单独的 `usize`，但它只有在 `&String` 的上下文中才是一个有意义的数字。换句话说，因为它是一个与 `String` 分离的值，无法保证它在未来仍然有效。请考虑示例 4-8 中的程序，它使用了示例 4-7 中的 `first_word` 函数。

<Listing number="4-8" file-name="src/main.rs" caption="存储调用 `first_word` 函数的结果，然后更改 `String` 的内容">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

这段程序编译时没有任何错误，而且在调用 `s.clear()` 之后使用 `word` 也不会有问题。因为 `word` 与 `s` 的状态完全没有关联，`word` 仍然包含值 `5`。我们可以用那个值 `5` 和变量 `s` 来尝试提取第一个单词，但这将是一个错误（Bug），因为自从我们将 `5` 保存到 `word` 中之后，`s` 的内容已经改变了。

担心 `word` 中的索引（Index）与 `s` 中的数据不同步是繁琐且容易出错的！如果我们编写一个 `second_word` 函数，管理这些索引（Index）会变得更加脆弱。它的签名必须像这样：

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

现在我们追踪的是起始_和_结束索引（Index），而且我们有更多的值是根据特定状态下的数据计算出来的，却完全没有与该状态绑定。我们有三个不相关的变量需要保持同步。

幸运的是，Rust 对此问题有一个解决方案：字符串切片（String Slice）。

### 字符串切片（String Slice）

*字符串切片（String Slice）* 是对 `String` 中一段连续元素的引用，它看起来像这样：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

`hello` 不是对整个 `String` 的引用，而是对 `String` 一部分的引用，通过额外的 `[0..5]` 部分来指定。我们使用方括号内的范围（Range）来创建切片（Slice），通过指定 `[starting_index..ending_index]`，其中 _`starting_index`_ 是切片（Slice）中的第一个位置，而 _`ending_index`_ 是切片（Slice）中最后一个位置的下一个位置。在内部，切片（Slice）的数据结构存储了起始位置和切片（Slice）的长度，该长度对应于 _`ending_index`_ 减去 _`starting_index`_。因此，对于 `let world = &s[6..11];` 的情况，`world` 将是一个切片（Slice），它包含一个指向 `s` 中索引 6 处字节的指针，长度值为 `5`。

图 4-7 以图表形式展示了这一点。

<img alt="三个表格：一个表格表示 s 的栈（Stack）数据，它指向堆（Heap）上字符串数据表格中索引 0 处的字节。第三个表格表示切片 world 的栈数据，其长度值为 5，并指向堆数据表格中的字节 6。"
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-7：引用 `String` 一部分的字符串切片（String Slice）</span>

使用 Rust 的 `..` 范围（Range）语法，如果你想从索引 0 开始，可以省略两个点之前的值。换句话说，以下是等价的：

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

同理，如果你的切片（Slice）包含 `String` 的最后一个字节，你可以省略尾部的数字。这意味着以下是等价的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

你也可以同时省略两个值来获取整个字符串的切片（Slice）。因此，以下是等价的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 注意：字符串切片（String Slice）的范围（Range）索引必须位于有效的 UTF-8 字符边界上。如果你尝试在多字节字符的中间创建字符串切片，程序将退出并报错。

有了这些信息，我们来重写 `first_word` 以返回一个切片（Slice）。表示"字符串切片"的类型写作 `&str`：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

我们使用与示例 4-7 相同的方式获取单词结尾的索引（Index）：寻找第一个出现的空格。当找到空格时，我们返回一个字符串切片（String Slice），以字符串的开头和空格的索引作为起始和结束索引。

现在当我们调用 `first_word` 时，我们返回一个与底层数据绑定的单一值。该值由指向切片（Slice）起始点的引用和切片中元素的数量组成。

返回切片（Slice）同样适用于 `second_word` 函数：

```rust,ignore
fn second_word(s: &String) -> &str {
```

现在我们有了一个更不易出错的直观 API，因为编译器会确保对 `String` 的引用保持有效。还记得示例 4-8 程序中的错误（Bug）吗？我们获取了第一个单词结尾的索引（Index），然后清空了字符串，导致索引失效？那段代码在逻辑上是错误的，但没有立即显示任何错误。如果我们继续尝试使用已清空字符串的第一个单词索引，问题会在之后暴露出来。切片（Slice）使这种错误（Bug）变得不可能，并让我们更早地知道代码存在问题。使用切片版本的 `first_word` 会抛出一个编译时错误：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

这是编译器错误：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

回顾一下借用规则（Borrowing Rules）：如果我们对某个值拥有不可变引用，就不能再获取可变引用。因为 `clear` 需要截断 `String`，它需要获取一个可变引用。而 `clear` 调用之后的 `println!` 使用了 `word` 中的引用，所以此时不可变引用必须仍然活跃。Rust 不允许 `clear` 中的可变引用和 `word` 中的不可变引用同时存在，因此编译失败。Rust 不仅让我们的 API 更易于使用，还在编译时消除了一整类错误！

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### 字符串字面量（String Literal）作为切片（Slice）

回想一下，我们提到过字符串字面量（String Literal）存储在二进制文件中。既然我们已经了解了切片（Slice），就可以恰当地理解字符串字面量了：

```rust
let s = "Hello, world!";
```

这里 `s` 的类型是 `&str`：它是一个指向二进制文件中该特定点的切片（Slice）。这也是字符串字面量不可变的原因；`&str` 是一个不可变引用。

#### 字符串切片（String Slice）作为参数

知道你可以获取字面量和 `String` 值的切片（Slice）后，我们对 `first_word` 有了进一步的改进，那就是它的签名：


```rust,ignore
fn first_word(s: &String) -> &str {
```

更有经验的 Rustacean 会编写如示例 4-9 所示的签名，因为它允许我们对 `&String` 值和 `&str` 值使用同一个函数。

<Listing number="4-9" caption="通过将 `s` 参数的类型改为字符串切片来改进 `first_word` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

如果我们有一个字符串切片（String Slice），可以直接传递它。如果我们有一个 `String`，可以传递 `String` 的切片或对 `String` 的引用。这种灵活性利用了解引用强制多态（Deref Coercion）的特性，我们将在第 15 章的["在函数和方法中使用解引用强制多态"][deref-coercions]<!-- ignore -->部分介绍。

定义一个接受字符串切片（String Slice）而不是 `String` 引用的函数，使我们的 API 更加通用和有用，同时没有丢失任何功能：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### 其他切片（Slice）

你可以想象，字符串切片（String Slice）是针对字符串的。但也有更通用的切片类型。考虑这个数组：

```rust
let a = [1, 2, 3, 4, 5];
```

就像我们可能想要引用字符串的一部分一样，我们可能想要引用数组的一部分。我们可以这样做：

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

这个切片（Slice）的类型是 `&[i32]`。它的工作方式与字符串切片相同，通过存储对第一个元素的引用和一个长度。你会在各种其他集合中使用这种切片。当我们谈到第 8 章的向量（Vector）时，我们将详细讨论这些集合。

## 总结

所有权（Ownership）、借用（Borrowing）和切片（Slice）这些概念确保了 Rust 程序在编译时的内存安全。Rust 语言像其他系统编程语言一样，让你能够控制内存使用。但是，当所有者（Owner）离开作用域时，数据的所有者会自动清理数据，这意味着你不需要为此编写和调试额外的代码。

所有权（Ownership）影响着 Rust 中许多其他部分的工作方式，因此我们将在本书的其余部分进一步讨论这些概念。让我们进入第 5 章，看看如何将数据片段组合到 `struct` 中。

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
