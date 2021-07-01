Trait core::marker::[Unpin][1]
---
*Stable 1.52.0*

## 签名

```rust
pub auto trait Unpin { }
```

## 注释

（标记）类型在被 pinned 后仍能被安全的移动。（解 pin）

Rust 本身没有不可移动的类型，并且认为移动（例如，通过赋值或者 [`mem::replace`][2]）总是安全的。

[`Pin`][3] 类型被引入用来防止类型系统中的移动。指针 `P<T>` 被包裹在 `Pin<P<T>>` 里能变得不可移动。有关 pin 的细节详情可以查看 [`Pin` 模块][4]。

给类型 `T` 实现 `Unpin` trait 可以解除该类型的移动限制，也就允许用 `mem::replace` 等函数将 `T` 从 `Pin<P<T>>` 中移动出来。

`Unpin` 对所有的非 pin 数据没有任何影响。实际上，`mem::replace` 也可以愉快地移动 `!Unpin` 数据（因为只要给定任何 `&mut T` 它就能工作，而非只对 `T: Unpin`）。只不过，因为不能获得 `&mut T` 所以你无法对包裹在 `Pin<P<T>>` 里的数据使用 `mem::replace`。

所以以下例子仅当在实现了 `Unpin` 的类型上起作用：

```rust
use std::mem;
use std::pin::Pin;

let mut string = "this".to_string();
let mut pinned_string = Pin::new(&mut string);

// 调用 `mem::replace` 需要提供一个可变引用。
// 可以通过（隐式地）调用 `Pin::deref_mut` 来获取这样的可变引用，
// 但是能这么做 `String` 全因为它实现了 `Unpin`。
mem::replace(&mut *pinned_string, "other".to_string());
```

该 trait 对几乎所有类型都是自动实现的。

## 拓展阅读

- [Unpin 的实现列表][5]
- [Rust 的 Pin 与 Unpin][6]


[1]: https://doc.rust-lang.org/core/marker/trait.Unpin.html
[2]: https://doc.rust-lang.org/core/mem/fn.replace.html
[3]: https://doc.rust-lang.org/core/pin/struct.Pin.html
[4]: https://doc.rust-lang.org/core/pin/index.html
[5]: https://doc.rust-lang.org/core/marker/trait.Unpin.html#implementors
[6]: https://folyd.com/blog/rust-pin-unpin/
