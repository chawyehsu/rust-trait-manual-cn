Trait core::marker::[Send][1]
---

## 签名

```rust
pub unsafe auto trait Send { }
```

## 注释

（标记）类型可以安全地跨线程传递所有权，即跨线程移动。

当编译器判定如果情况合适，该 trait 会被自动实现。

非 `Send` 类型的一个例子便是引用计数指针 [`rc::Rc`][2]。当两个线程尝试克隆指向同一个引用计数值的 Rc 指针时，它们可能同时在对引用计数进行更新，而这是[未定义行为][3]，因为 Rc 不使用原子操作。Rc 的同类 [`sync::Arc`][4] 使用原子操作（有一定的开销），所以它是 `Send` 类型。

更多细节可以查看[死灵书][5]。

## 拓展阅读

- [实现列表][6]
- [The Rustonomicon: Send and Sync][7]

[1]: https://doc.rust-lang.org/core/marker/trait.Send.html
[2]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[3]: https://doc.rust-lang.org/reference/behavior-considered-undefined.html
[4]: https://doc.rust-lang.org/std/sync/struct.Arc.html
[5]: https://doc.rust-lang.org/nomicon/send-and-sync.html
[6]: https://doc.rust-lang.org/core/marker/trait.Send.html#implementors
[7]: https://doc.rust-lang.org/nomicon/send-and-sync.html
