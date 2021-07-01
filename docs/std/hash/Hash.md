Trait std::hash::[Hash][1]
---
*Stable 1.52.0*

## 签名

```rust
pub trait Hash {
    pub fn hash<H>(&self, state: &mut H)
    where
        H: Hasher;

    pub fn hash_slice<H>(data: &[Self], state: &mut H)
    where
        H: Hasher,
    { ... }
}
```

表示一个可哈希的类型。

实现 `Hash` trait 的类型可以被一个 [`Hasher`][2] 实例进行哈希。

#### 实现 `Hash`

如果一个类型的所有字段都实现了 `Hash` 则可以通过 `#[derive(Hash)]` 对其派生 `Hash`。其哈希值等价于对其每个字段调用 [`hash`][3] 方法结果的合并。

```rust
#[derive(Hash)]
struct Rustacean {
    name: String,
    country: String,
}
```

如果你想控制哈希值的计算方式，那么可以为其手动实现 `Hash` trait：

```rust
use std::hash::{Hash, Hasher};

struct Person {
    id: u32,
    name: String,
    phone: u64,
}

impl Hash for Person {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.id.hash(state);
        self.phone.hash(state);
    }
}
```

#### `Hash` 与 `Eq`

同时实现了 `Hash` 和 [`Eq`][4] trait 时，保持以下的属性是很重要的:

```
k1 == k2 -> hash(k1) == hash(k2)
```

换言之，当两个键相同时，它们的哈希值务必也相同。[`HashMap`][5] 与 [`HashSet`][6] 均依赖此行为。

幸运的是，当使用 `#[derive(PartialEq, Eq, Hash)]` 来同时派生 `Eq` 和 `Hash` 时你不需要担心该属性的保持问题。

## 必需实现的方法

```rust
pub fn hash<H>(&self, state: &mut H)
where
    H: Hasher,
```

用提供的 `Hasher` 消费当前值。

#### 示例

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

let mut hasher = DefaultHasher::new();
7920.hash(&mut hasher);
println!("Hash is {:x}!", hasher.finish());
```

## 提供的方法

```rust
pub fn hash_slice<H>(data: &[Self], state: &mut H)    # MSRV 1.3.0
where
    H: Hasher,
```

用提供的 `Hasher` 消费提供的切片。

虽说该方法是为了方便，但是它的实现是没有明确规定的。它不保证其调用结果与分别重复调用 `hash` 方法的结果一致。如果提供的切片在 `PartialEq` 实现中不能被当做一个整体单元时，则应该使用 `hash` 方法。

#### 示例

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

let mut hasher = DefaultHasher::new();
let numbers = [6, 28, 496, 8128];
Hash::hash_slice(&numbers, &mut hasher);
println!("Hash is {:x}!", hasher.finish());
```

## 拓展阅读

- [Hash 的实现列表][7]


[1]: https://doc.rust-lang.org/std/hash/trait.Hash.html
[2]: https://doc.rust-lang.org/std/hash/trait.Hasher.html
[3]: https://doc.rust-lang.org/std/hash/trait.Hash.html#tymethod.hash
[4]: https://doc.rust-lang.org/std/cmp/trait.Eq.html
[5]: https://doc.rust-lang.org/std/collections/struct.HashMap.html
[6]: https://doc.rust-lang.org/std/collections/struct.HashSet.html
[7]: https://doc.rust-lang.org/std/hash/trait.Hash.html#implementors
