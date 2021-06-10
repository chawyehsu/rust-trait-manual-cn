Trait core::marker::[Sync][1]
---

## 签名

```rust
pub unsafe auto trait Sync { }
```

## 注释

（标记）类型可以安全地跨线程传递不可变引用，即跨线程共享。

当编译器判定如果情况合适，该特质会被自动实现。

该特质的精确定义是：当且仅当 `&T` 是 `Send` 时，`T` 才会是 `Sync`。换句话说，即在线程间传递 `&T` 引用时，不存在[未定义行为][2]（包括数据竞争）。

如预期，诸如 `u8` 和 `f64` 等基本类型都是 `Sync` 的，并且包含它们的简单聚合类型如元组、结构体和枚举都是 `Sync` 的。更多 `Sync` 的例子包括像“不可变”类型 `&T`，以及继承了可变性的 `Box<T>`，`Vec<T>` 和其它集合类型。（当然泛型参数本身需要是 `Sync` 时它们的容器才可以是 `Sync` 的）

一个出人意料的推论是，（当 `T` 是 `Sync` 时）`&mut T` 也是 `Sync` 的，虽然它看上去好像会造成可变性不同步。巧妙之处便是一个可变引用的不可变引用（即 `& &mut T`）是只读的，它看上去就像是 `& &T`。所以它并没有数据竞争的风险。

非线程安全下一些具备“内部可变性”的类型，比如 [`Cell`][3] 和 [`RefCell`][4] 则不是 `Sync` 的。这些类型允许通过其不可变引用来改变内部值。例如 `Cell<T>` 的 `set` 方法使用的是 `&self`，也就是只需要一个 `&Cell<T>`（即可调用 `set` 修改内部值）。该方法造成了不同步，所以 `Cell` 不能是 `Sync` 的。

另一个非 `Sync` 的类型的例子是引用计数指针 [`Rc`][5]。给定一个引用 `&Rc<T>`，可以克隆出一个新的 `Rc<T>`，进而以非原子操作的方式修改引用计数。

Rust 提供了[原子类型][6]与显式锁 [`sync::Mutex`][7] 和 [`sync::RwLock`][8] 来应对需要线程安全的内部可变性的用例。这些类型确保了任意修改操作不会造成数据竞争，所以它们是 `Sync` 的。类似地，[`sync::Arc`][9] 提供了一个线程安全版的 Rc。

同时，任何具有内部可变性的类型必定要使用 [`cell::UnsafeCell`][13] 来包裹其内部值以实现通过不可变引用来进行修改操作。如果不这么做会造成未定义行为。例如，用 [`transmute`][10] 将 `&T` 转换为 `&mut T` 是无效的。

`Sync` 的更多细节可以查看[死灵书][11]。

## 拓展阅读

- [Sync 的实现列表][12]
- [The Rustonomicon: Send and Sync][11]


[1]: https://doc.rust-lang.org/core/marker/trait.Sync.html
[2]: https://doc.rust-lang.org/reference/behavior-considered-undefined.html
[3]: https://doc.rust-lang.org/core/cell/struct.Cell.html
[4]: https://doc.rust-lang.org/core/cell/struct.RefCell.html
[5]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[6]: https://doc.rust-lang.org/core/sync/atomic/index.html
[7]: https://doc.rust-lang.org/std/sync/struct.Mutex.html
[8]: https://doc.rust-lang.org/std/sync/struct.RwLock.html
[9]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[10]: https://doc.rust-lang.org/core/mem/fn.transmute.html
[11]: https://doc.rust-lang.org/nomicon/send-and-sync.html
[12]: https://doc.rust-lang.org/core/marker/trait.Sync.html#implementors
[13]: https://doc.rust-lang.org/core/cell/struct.UnsafeCell.html
