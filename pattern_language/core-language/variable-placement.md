---
description: 将变量置于内存中的特定地址。
---

# Variable Placement

## Variable Placement

为了使运行时开始解码数据，需要在二进制数据中的某个位置放置变量。为此使用变量放置语法：

```rust
u32 myPlacedVariable @ 0x110;
```

这将创建一个名为`myPlacedVariable`的新无符号32位变量，并将其放置在地址`0x110`处。

运行时现在将把从偏移量`0x110`开始的4个字节视为u32类型，并据此解码该地址处的字节。

<figure><img src="../.gitbook/assets/placement/data.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/placement/hex.png" alt=""><figcaption></figcaption></figure>

变量定位不仅限于内置类型。所有类型，包括结构体、枚举、联合体等自定义类型，均可进行定位。


### Global variables

在模式运行期间，有时需要全局存储数据。此时可使用全局变量。其语法与局部变量相同，但末尾不包含_@_定位指令。

```rust
u32 globalVariable;
```

### Calculated pointers

相同的定位语法也可用于结构体内部，以指定模式在内存中的放置位置。这些变量不会增加其所处结构体的整体大小。
