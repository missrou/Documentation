---
说明: 内置基本数据类型
---

# Data Types

类型是定义特定内存区域应如何解释、格式化和显示的基本实体。

### Built-in Types

#### Unsigned Integers

无符号整数类型表示常规二进制数。它们以整数值形式显示，取值范围为 $$0$$ to $$(2^{8*Size}-1)$$.

| Name   | Size     |
| ------ | -------- |
| `u8`   | 1 Byte   |
| `u16`  | 2 Bytes  |
| `u24`  | 3 Bytes  |
| `u32`  | 4 Bytes  |
| `u48`  | 6 Bytes  |
| `u64`  | 8 Bytes  |
| `u96`  | 12 Bytes |
| `u128` | 16 Bytes |

#### Signed Integers

带符号整数类型采用二进制补码表示法表示带符号二进制数。它们以整数值的形式呈现，取值范围为 $$-(2^{8*Size}-1)$$ to $$(2^{8*Size}-1) - 1$$

| Name   | Size     |
| ------ | -------- |
| `s8`   | 1 Byte   |
| `s16`  | 2 Bytes  |
| `s24`  | 3 Bytes  |
| `s32`  | 4 Bytes  |
| `s48`  | 6 Bytes  |
| `s64`  | 8 Bytes  |
| `s96`  | 12 Bytes |
| `s128` | 16 Bytes |

#### Floating Points

浮点类型表示浮点数。在多数现代平台上，其遵循IEEE754标准，但并非绝对保证。

| Name     | Size                                   |
| -------- | -------------------------------------- |
| `float`  | Unspecified (4 Bytes, IEEE754 usually) |
| `double` | Unspecified (8 Bytes, IEEE754 usually) |

#### Special

| Name     | Size    | Description                                             |
| -------- | ------- | ------------------------------------------------------- |
| `char`   | 1 Byte  | ASCII Character                                         |
| `char16` | 2 Bytes | UTF-16 Wide Character                                   |
| `bool`   | 1 Byte  | Boolean value `true`/`false`                            |
| `str`    | Varying | 堆分配的字符串，仅可在函数中使用                           |
| `auto`   | Varying | 自动类型推断，仅限于函数内部使用                           |

### Endianness

默认情况下，所有内置类型均按本机字节序进行解释。这意味着若运行时环境运行于小端机型，所有类型将被视为小端序；若运行于大端机型，则被视为大端序。

不过，可以在全局、按类型或按变量的基础上覆盖此默认设置。只需在任何类型前添加前缀 `le`（表示小端序）或 `be`（表示大端序）即可：

```rust
le u32 myUnsigned;  // Little endian 32 bit unsigned integer
be double myDouble; // Big endian 64 bit double precision floating point
s8 myInteger;       // Native endian 8 bit signed integer
```

有关设置全局字节序的信息，请参阅[the endianness pragma](preprocessor.md#endian) .

### Literals

字面量是表示特定常量的固定值。可用的字面量如下：

| Type                    | Example             |
| ----------------------- | ------------------- |
| Decimal Integer         | `42`, `-1337`       |
| Unsigned 32 bit integer | `69U`               |
| Signed 32 bit integer   | `69`, `-123`        |
| Hexadecimal Integer     | `0xDEAD`            |
| Binary Integer          | `0b00100101`        |
| Octal Integer           | `0o644`             |
| Float                   | `1.414F`            |
| Double                  | `3.14159`, `1.414D` |
| Boolean                 | `true`, `false`     |
| Character               | `'A'`               |
| String                  | `"Hello World"`     |

### Enums

枚举是一种数据类型，由一组具有特定大小的命名常量组成。

枚举类型特别适用于为存储在内存中的值赋予含义。其定义方式与其他C类语言类似：枚举中的首个条目将关联值`0x00`，后续条目依次递增计数。若某个条目被显式赋予特定值，则该值之后的所有条目将从该值开始继续计数。

```rust
enum StorageType : u16 {
  Plain,    // 0x00
  Compressed = 0x10,
  Encrypted // 0x11
};
```

枚举名后冒号后的类型声明了枚举的底层类型，可以是任何内置数据类型。该类型仅影响枚举的大小。

<figure><img src="../.gitbook/assets/enums/data.png" alt=""><figcaption></figcaption></figure>

#### Enum Range

有时，一组数值可能指向同一个枚举值，此时枚举范围便能发挥作用。枚举范围会将指定范围内的所有值统一显示为该枚举项。在数学表达式中使用范围值时，其结果将为该范围的起始值。

```rust
enum NumberType : u16 {
  Unsigned      = 0x00 ... 0x7F,
  Signed        = 0x80,
  FloatingPoint = 0x90
};
```

### Arrays

数组是由一个或多个相同类型的值组成的连续集合。

#### Constant sized array

可通过在方括号内输入条目数量来指定常量大小。该值也可命名另一个变量，系统将读取该变量以获取大小。

```rust
u32 array[100] @ 0x00;
```

#### Unsized array

数组的大小可以留空，这种情况下它会持续增长，直到遇到一个全为零的条目为止。

```rust
char string[] @ 0x00;
```

#### Loop sized array

有时数组需要在满足特定条件时持续增长。以下数组将持续增长，直至遇到值为`0xFF`的字节。

```rust
u8 string[while(std::mem::read_unsigned($, 1) != 0xFF)] @ 0x00;
```

#### Optimized arrays

大型数组的计算耗时长且占用大量内存。因此，内置类型的数组会自动优化为仅创建一个数组类型的实例，并根据需要进行移动操作。

同样的优化也可用于自定义类型，只需为其添加`[[static]]`属性标记。但此操作仅适用于始终具有相同大小和内存布局的自定义类型。否则结果可能无效！

#### Strings

`char` 和 `char16` 类型在数组中使用时表现不同。它们不会显示为字符数组，而是以字符串形式呈现；如下例所示，字符串末尾以空字节终止。

```rust
char myCString[];
char16 myUTF16String[];
```

### Pointers

指针是一种变量，其值被视为地址，用于查找其所指向值的地址。

```rust
u16 *pointer : u32 @ 0x08;
```

此代码声明了一个指针，其地址类型为`u32`，指向一个`u16`类型变量。

```rust
u32 *pointerToArray[10] : s16 @ 0x10;
```

此代码声明了一个指向10个`u32`数组的指针，该指针的大小为`s16`。

该地址将始终被视为绝对地址。请确保正确设置数据的基址，以使指针按预期工作。

<figure><img src="../.gitbook/assets/pointers/data.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/pointers/hex.png" alt=""><figcaption></figcaption></figure>

### Bitfields

位域与结构体类似，但它们处理的是未对齐的独立位。可用于解码位标志或其他使用少于8位存储值的类型。

```rust
bitfield Permission {
  r : 1;
  w : 1;
  x : 1;
};
```

位域中的每个条目由字段名、冒号以及该字段的位数组成。单个字段最多可占用64位。

<figure><img src="../.gitbook/assets/bitfields/data.png" alt=""><figcaption></figcaption></figure>

位域也可嵌套使用，或作为数组置于其他位域内部。此时，对齐规则仅适用于最外层位域置于结构体类型内部的情况，而非位域内部。

#### Bitfield field types

默认情况下，每个位域字段都被解释为无符号值，但也可以将其解释为有符号数、布尔值或枚举值。

```cpp
bitfield TestBitfield {
    regular_value           : 4;    // Regular field, regular unsigned value
    unsigned unsigned_value : 5;    // Unsigned field, same as regular_value
    signed   signed_value   : 4;    // Signed field, interpreting the value as two's complement
    bool     boolean_value  : 1;    // Boolean field, 
    TestEnum enum_value     : 8;    // Enum field, displays the enum value corresponding to that value
};
```

此外，还可以在常规类型中交错使用位域字段。

```rust
bitfield InterleavedBitfield {
    field_1 : 4;
    field_2 : 2;
    u16 regular_value;
};
```
在位域中使用完整字段时，当前位偏移量总会被对齐到下一个完整字节边界。

#### Padding

也可以使用填充语法在字段之间插入填充内容。

```rust
bitfield Flags {
  a : 1;
  b : 2;
  padding : 4;
  c : 1;
};
```

这会在字段`b`和`c`之间插入4位填充。

### Structs

结构体是一种数据类型，它将多个变量捆绑成单一类型。

一个用于存储浮点数3D向量的非常简单的结构体可能如下所示：

```rust
struct Vector3f {
  float x, y, z;
};
```

使用定位语法将其放入内存时，结构体中的所有成员将从指定地址开始直接相邻放置。

<figure><img src="../.gitbook/assets/structs/data.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/structs/hex.png" alt=""><figcaption></figcaption></figure>

#### Padding

默认情况下，结构体成员之间不存在填充。由于这种情况并非总是理想，因此可通过`padding`关键字手动插入填充。

```rust
struct Vector3f {
  float x;
  padding[4];
  float y;
  padding[8];
  float z;
};
```

该代码将在成员`x`和`y`之间插入4字节填充，并在`y`和`z`之间插入8字节填充。

<figure><img src="../.gitbook/assets/structs/padding.png" alt=""><figcaption></figcaption></figure>

#### Inheritance

继承允许将父结构体的所有成员复制到子结构体中，使其在子结构体中可用。

```rust
struct Parent {
  u32 type;
  float value;
};

struct Child : Parent {
  char string[];
};
```

结构体 `Child` 现在包含 `type`、`value` 和 `string`。

#### Anonymous members

在结构体或联合体内声明变量时，可以不为其命名。当你知道某个偏移量处存在特定类型的模式，但变量名称尚未确定或不重要时，这种方式就很有用。

```rust
struct MyStruct {
  u32;
  float;
  MyOtherStruct;
};
```

#### Conditional parsing

模式语言提供了高级特性，允许定义更为复杂的结构，这些特性在以下文档中有详细说明： [Control Flow](control-flow.md) .

### Unions

联合体与结构体类似，它们将多个变量捆绑成一种新类型，但这些变量并非连续放置，而是共享相同的起始地址。

这有助于解释和检查多种不同类型的数据，如以下所示：

```rust
union Converter {
  u32 integerData;
  float floatingPointData;
};
```

<figure><img src="../.gitbook/assets/unions/data.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/unions/hex.png" alt=""><figcaption></figcaption></figure>

### Using declarations

使用声明可为现有类型赋予新名称，并可选地为其添加额外限定符。以下代码创建了一个名为`Offset`的新类型，该类型为大端序32位无符号整数。现在它可替代任何其他类型使用。

```rust
using Offset = be u32;
```

#### Forward declaration

当存在两种类型相互递归引用时，必须对其中一种类型进行前向声明，以便在运行时需要时能够识别所有类型。

这可以通过 `using TypeName;` 语法实现。

```rust
// 告知语言系统未来将存在名为B的类型，因此当它遇到
// 具有此类型的变量，必须推迟解析直至该类型被声明
using B;

struct A {
  bool has_b;

  if (has_b)
    B b;
};

struct B {
  bool has_a;

  if (has_a)
    A a;
};
```

### Templates

模板可用于将自定义类型的成员类型替换为占位符，这些占位符可在后续实例化该类型时进行定义。

模板可用于`struct`、`union`和`using`声明：

```rust
struct MyTemplateStruct<T> {
  T member;
};

union MyTemplateStruct<Type1, Type2> {
  Type1 value1;
  Type2 value2;
};

using MyTemplateUsing<Type1> = MyTemplateStruct<Type1, u32>;
```

这些模板随后可用于创建具体类型：

```rust
MyTemplateStruct<u32, u64> myConcreteStruct @ 0x00;
```

#### Non-Type Template Parameters

同样可以使用模板将表达式传递给类型。例如数字、字符串或变量（包括自定义类型）。

要将模板参数标记为非类型模板参数，请使用 `auto` 关键字。

```rust
struct Array<T, auto Size> {
  T data[Size];
};

Array<u32, 0x100> array @ 0x00;
```

#### Pattern local variables

在模式内部可以声明局部变量，这些变量不会出现在最终类型中，但可用于存储后续使用的信息。声明局部变量时，只需使用`=`运算符为其赋值即可。

```rust
struct MyType {
    u32 x, y, z; // Regular members
    float localVariable = 0.5; // Local variable
};
```
