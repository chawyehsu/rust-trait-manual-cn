Trait core::marker::[Copy][1]
---

## 签名

```rust
pub trait Copy: Clone { }
```

## 注释

（标记）类型通过拷贝其位数据即可复制其值。

默认情况下，变量绑定具有“移动语义”。也即：

```rust
#[derive(Debug)]
struct Foo;

let x = Foo;

let y = x;

// `x` 已经移动到 `y`，不能再被使用

// println!("{:?}", x); // error: use of moved value
```

不过，如果类型实现了 `Copy`，那么它就具有“拷贝语义”：

```rust
// 这里我们直接派生 `Copy`。因为 `Copy` 是 `Clone` 的子 trait，所以也需要派生 `Clone`。
#[derive(Debug, Copy, Clone)]
struct Foo;

let x = Foo;

let y = x;

// `y` 是 `x` 的拷贝

println!("{:?}", x); // A-OK!
```

值得注意的是，在这两个例子中唯一的不同点是在赋值之后还能否访问 `x` 变量。因为本质上，复制和移动都会导致内存中的位数据被复制，尽管有时候这会被优化掉。

### 如何实现 `Copy`？

有两种方法给你的类型实现 `Copy`。最简单的是使用 `derive`：

```rust
#[derive(Copy, Clone)]
struct MyStruct;
```

也可以手动分别实现 `Copy` 和 `Clone`：

```rust
struct MyStruct;

impl Copy for MyStruct { }

impl Clone for MyStruct {
    fn clone(&self) -> MyStruct {
        *self
    }
}
```

这两种方法之间有个不同点：`derive` 方法会同时给类型的参数也加上 `Copy`，这并不适用于所有情况。

### `Copy` 与 `Clone` 的区别是什么？

拷贝是隐式发生的，比如出现在 `y = x` 这个例子中。`Copy` 的行为是不可以被重载的，它总是按位进行复制。

克隆是显式发生的，如 `x.clone()`。在实现 [`Clone`][2] 时可以为其指定任意类型独立的行为来做到安全地复制值。例如，[`String`][3] 的 `Clone` 实现是将指针指向的字符串缓冲区复制到堆上。如果对 `String` 进行简单的按位复制的话，则仅仅只是复制了它的指针，这会造成后续指针的双重释放。所以，`String` 是可 `Clone` 的而不能是可 `Copy` 的。

`Clone` 是 `Copy` 的父 trait，所以任何实现了 `Copy` 的类型必然实现了 `Clone`。如果一个类型是可 `Copy` 的，那么它的 `Clone` 实现就只需要返回 `*self`（见上文的例子）。

### 什么情况下类型可实现 `Copy`？

一个类型如果它的所有部分都实现了 `Copy`，那么它本身也可以实现 `Copy`。例如，以下结构体实现 `Copy`：

```rust
#[derive(Copy, Clone)]
struct Point {
   x: i32,
   y: i32,
}
```

`i32` 是可 `Copy` 的，这样整个结构体可 `Copy`，所以 `Point` 也就可实现 `Copy` 了。作为对比 ，考虑以下例子：

```rust
struct PointList {
    points: Vec<Point>,
}
```

因为 `Vec<T>` 不是可 `Copy` 的，所以 `PointList` 结构体也就不能实现 `Copy`。当尝试给其添加 `Copy` 派生实现时，会出现错误：

```
the trait `Copy` may not be implemented for this type; field `points` does not implement `Copy`
```

不可变引用 `&T` 也是可 `Copy` 的，也就是说即使类型 `T` 本身不可 `Copy`，它的不可变引用也能是可 `Copy` 的。考虑以下结构体，它能实现 `Copy`，因为它拥有的只是上文不可 `Copy` 的 `PointList` 类型的一个不可变引用：

```rust
#[derive(Copy, Clone)]
struct PointListWrapper<'a> {
    point_list_ref: &'a PointList,
}
```

### 什么情况下类型不能实现 `Copy`？

某些类型无法安全地被拷贝。例如，拷贝 `&mut T` 会创建一个可变引用的别名。直接拷贝 `String` 会引发对 `String` 缓冲区的重复管理，进而导致双重释放。

从后一个例子进一步推论，任何实现了 [`Drop`][4] 的类型都不能是 `Copy` 的，因为除自身的字节外，它们还管理着而外的资源。

如果你试图对一个包含非 `Copy` 数据的结构体实现 `Copy`，你会遇到 [`E0204`][5] 错误。

### 什么情况下类型应该实现 `Copy`？

通常来说，如果你的类型可以实现 `Copy`，那它就应该实现。但是请注意，实现 `Copy` 会是你的类型的公共 API 的一部分。所以如果类型在未来可能要变成非 `Copy` 的话，那么最好现在就忽略实现 `Copy`，以免产生 API 的重要改动。

### 额外的实现

除了[实现列表][6]里提及的实现外，以下类型也实现了 `Copy`：

- 函数项类型（即，每个函数在定义时对应的特有类型）
- 函数指针（例如，`fn() -> i32`）
- 数组，任意大小，仅当数组项的类型也实现了 `Copy` 的时候（例如，`[i32; 123456]`）
- 元组，仅当元组项的类型也实现了 `Copy` 的时候（例如，`()`，`(i32, bool)`）
- 闭包，仅当其没有从外部环境捕获数据或者捕获的所有数据都实现了 `Copy` 的时候。注意捕获到的不可变引用总是实现了 `Copy` 的（即使被引用的类型本身没有实现 `Copy`），而捕获到的可变引用则总是没有实现 `Copy` 的。

## 拓展阅读

- [Copy 的实现列表][6]
- [The Rust Reference：函数项类型][7]
- [The Rust Reference：函数指针][8]


[1]: https://doc.rust-lang.org/core/marker/trait.Copy.html
[2]: https://doc.rust-lang.org/core/clone/trait.Clone.html
[3]: https://doc.rust-lang.org/std/string/struct.String.html
[4]: https://doc.rust-lang.org/core/ops/trait.Drop.html
[5]: https://doc.rust-lang.org/error-index.html#E0204
[6]: https://doc.rust-lang.org/core/marker/trait.Copy.html#implementors
[7]: https://doc.rust-lang.org/reference/types/function-item.html
[8]: https://doc.rust-lang.org/reference/types/function-pointer.html
