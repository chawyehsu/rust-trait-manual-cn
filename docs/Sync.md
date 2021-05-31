Trait core::marker::[Sync][1]
---

## 签名

```rust
pub unsafe auto trait Sync { }
```

（标记）类型可以安全地跨线程传递不可边引用，即跨线程共享。

当编译器判定如果情况合适，该 trait 会被自动实现。

该 trait 的精确定义是：当且仅当 `&T` 是 `Send` 时，`T` 才会是 `Sync`。换句话说，即在线程间传递 `&T` 引用时，不存在[未定义行为][2]（包括数据竞争）。

如预期，诸如 `u8` 和 `f64` 等基本类型都是 `Sync` 的，并且包含它们的简单聚合类型如元组、结构体和枚举都是 `Sync` 的。更多 `Sync` 类型的例子包括像“不可变”类型 `&T`，以及继承了可变性的 `Box<T>`，`Vec<T>` 和其它集合类型。（当然泛型参数本身需要是 `Sync` 时它们的容器才可以是 `Sync` 的）

一个出人意料的推论是，（当 `T` 是 `Sync` 时）`&mut T` 也是 `Sync` 的，虽然它看上去好像会造成可变性不同步。巧妙之处便是一个可变引用的不可变引用（即 `& &mut T`）是只读的，它看上去就像是 `& &T`。所以它并没有数据竞争的风险。

非线程安全下一些具备“内部可变性”的类型，比如 [`Cell`][3] 和 [`RefCell`][4] 则不是 `Sync` 的。这些类型允许通过其不可变引用来改变内部值。例如 `Cell<T>` 的 `set` 方法使用的是 `&self`，也就是只需要一个 `&Cell<T>`（即可调用 `set` 修改内部值）。该方法造成了不同步，所以 `Cell` 不能是 `Sync` 的。


[1]: https://doc.rust-lang.org/core/marker/trait.Sync.html
[2]: https://doc.rust-lang.org/reference/behavior-considered-undefined.html
[3]: https://doc.rust-lang.org/core/cell/struct.Cell.html
[4]: https://doc.rust-lang.org/core/cell/struct.RefCell.html
