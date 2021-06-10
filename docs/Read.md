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
        ...
    }

    #[unstable(feature = "can_vector", issue = "69941")]
    fn is_read_vectored(&self) -> bool {
        ...
    }

    #[unstable(feature = "read_initializer", issue = "42788")]
    #[inline]
    unsafe fn initializer(&self) -> Initializer {
        ...
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize> {
        ...
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn read_to_string(&mut self, buf: &mut String) -> Result<usize> {
        ...
    }

    #[stable(feature = "read_exact", since = "1.6.0")]
    fn read_exact(&mut self, buf: &mut [u8]) -> Result<()> {
        ...
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn by_ref(&mut self) -> &mut Self
    where
        Self: Sized,
    {
        ...
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn bytes(self) -> Bytes<Self>
    where
        Self: Sized,
    {
        ...
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn chain<R: Read>(self, next: R) -> Chain<Self, R>
    where
        Self: Sized,
    {
        ...
    }

    #[stable(feature = "rust1", since = "1.0.0")]
    fn take(self, limit: u64) -> Take<Self>
    where
        Self: Sized,
    {
        ...
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

[`File`][5] 实现了 `Read`：

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

因为 [`&[u8]`][6] 实现了 `Read` 所以可以从 [`&str`][7] 中读取：

```rust
use std::io::prelude::*;

fn main() -> io::Result<()> {
    let mut b = "This string will be read".as_bytes();
    let mut buffer = [0; 10];

    // 读取最多 10 字节
    b.read(&mut buffer)?;

    // 跟读 File 一样！
    Ok(())
}
```

## 必需实现的方法

```rust
fn read(&mut self, buf: &mut [u8]) -> Result<usize>
```

从当前源中读取字节到提供的缓存区中，并返回读取了多少字节。

该方法不保证是否阻塞以等待数据，但是如果对象需要阻塞读取却无法进行，那么它会返回一个 [`Err`][8] 值。

当该方法返回值是 [`Ok(n)`][9] 时，该方法的实现必须保证 `0 <= n <= buf.len()`。 `n` 非零时表示从当前源中读取了 `n` 字节到缓冲区 `buf` 中。当 `n` 值为零时，它表示以下两种情况中的一种：

1. 读取器已经读到了当前“文件尾”，无法再读取更多字节。注意这并不表示该读取器*永远*无法再读取更多字节。
2. 指定缓冲区的大小为 0。

即使读取器还未读到流的尾部时，出现返回值 `n` 小于缓冲区大小也并不是一个错误。因为出现这种情况可能是由于当前只剩少量字节可被读取（比如读到接近文件尾时），亦或者是外部信号中断了 `read()`。

实现该特质时实现成 `n > buf.len()` 是安全的，所以调用方不能依赖 `n <= buf.len()` 作为安全性判断。当用不安全的函数来访问读取的字节时，需要特别地小心。调用方必须确保即使 `n > buf.len()` 也不会出现未经检查的越界访问。

调用该方法时由于提供的缓冲区 `buf` 不会对其内容有任何保证，所以实现该方法时不能依赖 `buf` 中的任何内容属性。建议*实现*只往 `buf` 中写数据s而不读取其内容。

相应的，*调用方*也不应假设该方法的实现对如何使用 `buf` 有任何的保证。在实现该特质时，本是写缓冲区的代码实现成读缓冲区，这是有可能的，对实现也是安全的。因此在调用 `read()` 前作为调用方你有责任确保 `buf` 已被初始化。用一个（通过 [`MaybeUninit<T>`][10] 获取到的）未初始化的 `buf` 来调用 `read()` 是不安全的，可能会导致未定义行为。

#### 错误

当该方法遇到任意类型的 I/O 或其他错误时，它将返回错误。当返回错误时，必须保证没有任何字节已被读取。

[`ErrorKind::Interrupted`][11] 类型错误是非致命的，如果没有其它操作那么应该重新尝试读取操作。

#### 示例

[`File`][5] 实现了 `Read`：

```rust
use std::io;
use std::io::prelude::*;
use std::fs::File;

fn main() -> io::Result<()> {
    let mut f = File::open("foo.txt")?;
    let mut buffer = [0; 10];

    // 读取最多 10 字节
    let n = f.read(&mut buffer[..])?;

    println!("The bytes: {:?}", &buffer[..n]);
    Ok(())
}
```

## 提供的方法

```rust
fn read_vectored(&mut self, bufs: &mut [IoSliceMut<'_>]) -> Result<usize>    # MSRV 1.36.0
```

与 `read` 类似，不过是接收一个缓冲区切片来进行读取操作。

会按顺序将数据复制写入每一个缓冲区中，最后一个缓冲区有可能只被写入了部分数据。该方法的行为必须跟用一组串联起来的缓冲区单次调用 `read` 方法的行为一致。

默认实现会选出提供的切片中第一个非空缓冲区来调用 `read`，如果没有非空的则选出第一个空缓冲区进行调用。

```rust
fn is_read_vectored(&self) -> bool    # Unstable
```



```rust
unsafe fn initializer(&self) -> Initializer    # Unstable
```



```rust
fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize>
```


[1]: https://doc.rust-lang.org/std/io/trait.Read.html
[2]: https://doc.rust-lang.org/std/io/index.html
[3]: https://doc.rust-lang.org/std/io/struct.BufReader.html
[4]: https://doc.rust-lang.org/std/io/trait.BufRead.html
[5]: https://doc.rust-lang.org/std/fs/struct.File.html
[6]: https://doc.rust-lang.org/nightly/std/primitive.slice.html
[7]: https://doc.rust-lang.org/nightly/std/primitive.str.html
[8]: https://doc.rust-lang.org/std/result/enum.Result.html#variant.Err
[9]: https://doc.rust-lang.org/std/result/enum.Result.html#variant.Ok
[10]: https://doc.rust-lang.org/std/mem/union.MaybeUninit.html
[11]: https://doc.rust-lang.org/std/io/enum.ErrorKind.html#variant.Interrupted
