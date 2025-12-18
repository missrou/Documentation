# Control flow

模式语言提供了多种方式，能够根据已解析的值实时调整解析器的行为，从而让您能够相对轻松地解析复杂格式。

### Conditionals

在模式语言中，并非所有结构体都需要相同大小或相同布局。根据其他变量的值，可以动态改变要声明的变量。

```rust
enum Type : u8 {
    A = 0x54,
    B = 0xA0,
    C = 0x0B
};

struct PacketA {
    float x;
};

struct PacketB {
    u8 y;
};

struct PacketC {
    char z[];
};

struct Packet {
    Type type;

    if (type == Type::A) {
        PacketA packet;
    }
    else if (type == Type::B) {
        PacketB packet;
    }
    else if (type == Type::C)
        PacketC packet;
};

Packet packet[3] @ 0xF0;
```

该代码通过检查每个`Packet`的首字节来确定其类型，从而能够根据`PacketA`、`PacketB`和`PacketC`中定义的正确类型，相应地解码数据包的主体内容。

此类条件语句可用于结构体、联合体和位域中。

<figure><img src="../.gitbook/assets/conditionals/data.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/conditionals/hex.png" alt=""><figcaption></figcaption></figure>

### Match statements

匹配语句是条件语句的更强大替代方案。它们能更轻松地匹配多个值，并提供更多形式的比较逻辑。

```rust
enum Type : u8 {
    A = 0x54,
    B = 0xA0,
    C = 0x0B
};

struct PacketA {
    float x;
};

struct PacketB {
    u8 y;
};

struct PacketC {
    char z[];
};

struct Packet {
    Type type;

    match (type) {
        (Type::A): PacketA packet;
        (Type::B): PacketB packet;
        (Type::C): PacketC packet;
    }
};

Packet packet[3] @ 0xF0;
```

但匹配语句的功能远不止简单的开关。它还允许同时匹配多个值，并使用更复杂的比较逻辑。此外，`_` 通配符可匹配任意值，因此也构成了默认情况。

```rust
struct Packet {
  Type type;
  u8 size;

  match (type, size) {
    (Type::A, 0x200): PacketA packet;
    (Type::C, _): PacketC packet;
    (_, _): PacketB packet;
  }
};

Packet packet[3] @ 0xF0;
```

此外，匹配语句还具备特殊比较功能，支持更批量化的比较操作。`_…_` 运算符可用于匹配数值范围，而 `|` 运算符则支持在多个数值间进行匹配。

```rust
struct Packet {
  Type type;
  u8 size;

  match (type, size) {
    (Type::A, 0x200): PacketA packet;
    (Type::C, 0x100 | 0x200): PacketC packet;
    (Type::B, 0x100 ... 0x300): PacketB packet;
  }
};

Packet packet[3] @ 0xF0;
```

### Pattern control flow

条件解析最基础的形式是数组控制流语句`break`和`continue`。它们允许你根据当前解析项实例中的条件，停止对数组的解析或跳过元素。

#### Break

当遇到分隔符时，当前数组创建过程将终止。这意味着数组将保留所有已解析的条目（包括当前正在处理的条目），但不会继续扩展——即使请求的条目数量尚未达到。

```rust
struct Test {
  u32 x;

  if (x == 0x11223344)
    break;
};

// This array requests 1000 entries but stops growing as soon as it hits a u32 with the value 0x11223344
// causing it to have a size less than 1000
Test tests[1000] @ 0x00;
```

`break` 也可用于常规模式中，用于提前终止当前模式的解析。

若包含`break`的模式嵌套在另一个模式中，则仅当前模式的评估被终止，并在当前模式定义之后继续执行父结构中的代码。

#### Continue

当遇到继续语句时，当前评估的数组项会被计算以确定下一个数组项的偏移量，但随后会被丢弃。这可用于有条件地排除某些无效数组项或不应显示在模式数据列表中的项，同时仍能扫描数组覆盖的整个范围。

例如，这可以与 [In/Out Variables](in-out-variables.md) 结合使用，轻松过滤数组项。

```rust
struct Test {
  u32 value;

  if (value == 0x11223344)
    continue;
};

// This array requests 1000 entries but skips all entries where x has the value 0x11223344
// causing it to have a size less than 1000
Test tests[1000] @ 0x00;
```

`continue` 也可用于常规模式中，以完全跳过该模式。

若包含`continue`的模式嵌套在另一个模式中，则仅当前模式被丢弃，评估将在当前模式定义后的父结构体中继续进行。

#### The Searcher Pattern

搜索器模式是一种有用的设计模式，用于将结构体模式放置在动态确定（搜索到的）偏移量处。它结合了覆盖搜索区域的数组（干草堆），以及在找到目标模式后将其放置在目标位置的包装结构体。

```rust
struct Command {
  u32 data;
};

u8 needle = 0xCD;

struct PatternSearcher {
  u32 test = std::mem::read_unsigned($, 1);
  if (test != needle) { $ += 1; continue; }
  Command command @ $;
  $ += sizeof(Command);
};

// Search from 0x0000 to 0xFFFF
PatternSearcher search[while($ < 0xFFFF)] @ 0x00;

// Search the entire file
PatternSearcher search[while(!std::mem::eof())] @ 0x00;

// Search from 0x3FFF to the end of the file
PatternSearcher search[while(!std::mem::eof())] @ 0x3FFF;
```

### Return statements

函数外部的返回语句可用于提前终止当前程序的执行。

评估在执行`return`语句的位置停止。在此之前已评估的所有模式都将被处理完毕并存入内存，随后执行才终止。

### Try-Catch statements

Try-Catch代码块用于尝试放置可能出错的模式，然后处理该错误。

以下代码将尝试将所有模式放入`try`代码块中。若在此过程中发生任何错误，则会丢弃尝试放置的模式，将光标回退至`try`代码块开头的位置，并转而执行`catch`代码块。

```rust
struct Test {
    try {
        u32 x;
        SomeStructThatWillError someStruct;
    } catch {
        SomeAlternativeStruct someStruct;
    }
};
```
