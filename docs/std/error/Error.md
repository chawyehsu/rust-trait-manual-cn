Trait std::error::[Error][1]
---
*Stable 1.52.0*

## 签名

```rust
pub trait Error: Debug + Display {
    fn source(&self) -> Option<&(dyn Error + 'static)> { ... }
    fn backtrace(&self) -> Option<&Backtrace> { ... }
    fn description(&self) -> &str { ... }
    fn cause(&self) -> Option<&dyn Error> { ... }
}
```

`Error` trait 用于表示对错误值，即 [`Result<T, E>`][2] 中的 `E` 的基本期望。

错误值必须实现 `Display` 及 `Debug` trait 来描述它们自身。错误消息通常为简明小写且没有结束标点符号的句子。

```rust
let err = "NaN".parse::<u32>().unwrap_err();
assert_eq!(err.to_string(), "invalid digit found in string");
```

错误值可以提供原因链信息。


[1]: https://doc.rust-lang.org/std/error/trait.Error.html
[2]: https://doc.rust-lang.org/std/result/enum.Result.html
