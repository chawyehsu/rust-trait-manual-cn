Trait core::marker::[Sized][1]
---

## 签名

```rust
pub trait Sized {}
```

## 注释

（标记）类型在编译期可以确定大小。

所有类型参数都隐性地具有 `Sized` 约束，可以使用 `?Sized` 这个特殊语法来移除该约束。

```rust
struct Foo<T>(T);
struct Bar<T: ?Sized>(T);

// struct FooUse(Foo<[i32]>); // error: Sized is not implemented for [i32]
struct BarUse(Bar<[i32]>); // OK
```

有一个例外是特质中的 `Self` 类型。特质并不隐性地拥有 `Sized` 约束，因为该约束与[特质对象][2]不兼容。根据定义，一个特质需要代表其所有可能的实现，所以它可以是任意大小。

```rust
trait Foo { }
trait Bar: Sized { }

struct Impl;
impl Foo for Impl { }
impl Bar for Impl { }

let x: &dyn Foo = &Impl;    // OK
// let y: &dyn Bar = &Impl; // error: the trait `Bar` cannot
                            // be made into an object
```

## 拓展阅读

- [Sizedness in Rust][3]
- [RFC 0255: object safety][4]


[1]: https://doc.rust-lang.org/core/marker/trait.Sized.html
[2]: https://doc.rust-lang.org/book/ch17-02-trait-objects.html
[3]: https://github.com/pretzelhammer/rust-blog/blob/master/posts/sizedness-in-rust.md
[4]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
