Trait std::io::[Read][1]
---

## 签名

```rust
#[stable(feature = "rust1", since = "1.0.0")]
pub trait Read {
    #[stable(feature = "rust1", since = "1.0.0")]
    fn read(&mut self, buf: &mut [u8]) -> Result<usize>;

    #[stable(feature = "iovec", since = "1.36.0")]
    fn read_vectored(&mut self, bufs: &mut [IoSliceMut<'_>]) -> Result<usize> {
        default_read_vectored(|b| self.read(b), bufs)
    }

    #[unstable(feature = "can_vector", issue = "69941")]
    fn is_read_vectored(&self) -> bool {
        false
    }

    #[unstable(feature = "read_initializer", issue = "42788")]
    #[inline]
    unsafe fn initializer(&self) -> Initializer {
        Initializer::zeroing()
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize> {
        read_to_end(self, buf)
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn read_to_string(&mut self, buf: &mut String) -> Result<usize> {
        append_to_string(buf, |b| read_to_end(self, b))
    }

    #[stable(feature = "read_exact", since = "1.6.0")]
    fn read_exact(&mut self, buf: &mut [u8]) -> Result<()> {
        default_read_exact(self, buf)
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn by_ref(&mut self) -> &mut Self
    where
        Self: Sized,
    {
        self
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn bytes(self) -> Bytes<Self>
    where
        Self: Sized,
    {
        Bytes { inner: self }
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn chain<R: Read>(self, next: R) -> Chain<Self, R>
    where
        Self: Sized,
    {
        Chain { first: self, second: next, done_first: false }
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn take(self, limit: u64) -> Take<Self>
    where
        Self: Sized,
    {
        Take { inner: self, limit }
    }
}
```

## 注释

`Read` 特质允许从给定源中读取字节。

`Read` 特质的实现被叫做“读取器”。

定义读取器只需要实现一个必需的方法 `read()`。对 `read()` 的每一次调用将会尝试从给定的源中读取字节到提供的缓存区。其它方法都是根据 `read()` 实现的，这使得实现器只需要实现一个方法，即可提供多种方式读取字节。

读取器可以彼此进行组合。[`std::io`][2] 里的许多实现器会读取或提供实现了 `Read` 特质的类型。

请注意对 `read()` 的每一次调用会触发一次系统调用，所以，使用诸如 [`BufReader`][3] 等实现了 [`BufRead`][4] 特质的读取器将会更高效。

### 示例

`File` 实现了 `Read`：

```rust
use std::io;
use std::io::prelude::*;
use std::fs::File;

fn main() -> io::Result<()> {
    let mut f = File::open("foo.txt")?;
    let mut buffer = [0; 10];

    // 读取 10 字节
    f.read(&mut buffer)?;

    let mut buffer = Vec::new();
    // 读取整个文件
    f.read_to_end(&mut buffer)?;

    // 可以直接读取到 String，不需要再进行转换。
    let mut buffer = String::new();
    f.read_to_string(&mut buffer)?;

    // 查看 Read 特质源码了解更多其它方法。
    Ok(())
}
```

[1]: https://doc.rust-lang.org/std/io/trait.Read.html
[2]: https://doc.rust-lang.org/std/io/index.html
[3]: https://doc.rust-lang.org/std/io/struct.BufReader.html
[4]: https://doc.rust-lang.org/std/io/trait.BufRead.html
