# 表达式

### 操作符

| Operator      | Description                                                                   |
| ------------- | ----------------------------------------------------------------------------- |
| `a + b`       | 加                                                                            |
| `a - b`       | 减                                                                            |
| `a * b`       | 乘                                                                            |
| `a / b`       | 除                                                                            |
| `a % b`       | 求余                                                                          |
| `a >> b`      | 右移位                                                                        |
| `a << b`      | 左位移                                                                        |
| `~a`          | 位运算非                                                                      |
| `a & b`       | 位与运算                                                                      |
| `a \| b`      | Bitwise OR                                                                    |
| `a ^ b`       | Bitwise XOR                                                                   |
| `a == b`      | Equality comparison                                                           |
| `a != b`      | Inequality comparison                                                         |
| `a > b`       | Greater-than comparison                                                       |
| `a < b`       | Less-than comparison                                                          |
| `a >= b`      | Greater-than-or-equals comparison                                             |
| `a <= b`      | Less-than-or-equals comparison                                                |
| `!a`          | Boolean NOT                                                                   |
| `a && b`      | Boolean AND                                                                   |
| `a \|\| b`    | Boolean OR                                                                    |
| `a ^^ b`      | Boolean XOR                                                                   |
| `a ? b : c`   | Ternary                                                                       |
| `(a)`         | Parenthesis                                                                   |
| `function(a)` | [Function](functions.md) call |

`a`、`b` 和 `c` 可以是任何数字字面量或另一个表达式。

### Type Operators

类型运算符是对类型进行操作的运算符。它们只能用于变量，不能用于数学表达式。

| Operator        | Description                                  |
| --------------- | -------------------------------------------- |
| `addressof(a)`  | Address of variable                          |
| `sizeof(a)`     | Size of variable                             |
| `typenameof(a)` | String representation of the variable's type |

`a` 可以是变量，既可以通过直接命名，也可以通过成员访问来获取。

**PROVIDER OPERATORS**

`a` 也可以用 `$` 运算符替代，用于查询已加载数据的相关信息。

| Operator       | Description                     |
| -------------- | ------------------------------- |
| `addressof($)` | Base address of the loaded data |
| `sizeof($)`    | Size of the loaded data         |

### String Operators

字符串运算符是指直接作用于字符串的任何运算符。

| Operator       | Description                    |
| -------------- | ------------------------------ |
| `a + b`        | String concatenation           |
| `str * number` | String repetition              |
| `a == b`       | Lexical equality               |
| `a != b`       | Lexical inequality             |
| `a > b`        | Lexical greater-than           |
| `a >= b`       | Lexical greater-than-or-equals |
| `a < b`        | Lexical less-than              |
| `a <= b`       | Lexical less than-or-equals    |

### Member Access

成员访问是指访问结构体、联合体或位域内部的成员，或通过数组索引访问其值的行为。

以下展示最基础的操作，但它们可根据需要无限组合与扩展。

| Operation       | Access type                                                                           |
| --------------- | ------------------------------------------------------------------------------------- |
| `structVar.var` | Accessing a variable inside a struct, union or bitfield                               |
| `arrayVar[x]`   | Accessing a variable inside an array                                                  |
| `parent.var`    | Accessing a variable inside the parent struct or union of the current struct or union |
| `this`          | Referring to the current pattern. Can only be used inside of a struct or union        |

`parent` 操作指的是调用者结构体或联合体，而非继承部分中的“父结构体”。它也可用于函数中：

```rust
struct Field {
    u8 x;
};
fn rename_field() {
    std::core::set_display_name(parent.fields[0], "version");
};
struct Struct {
    Field fields[3];
    rename_field();
};
Struct s @ 0;  // Pattern Data shows s > fields > version
```


### `$` Dollar Operator

`$`是一种特殊运算符，它展开为当前模式内的当前偏移量。

```rust
#pragma base_address 0x00

std::print($); // 0
u32 x @ 0x00;
std::print($); // 4
```

也可以为`$`赋值来改变当前光标位置。

```rust
$ += 0x100;
```

`$`也可用于访问主数据的单个字节。

```rust
std::print($[0]); // Prints the value of the byte at address 0x00
```

### Casting Operator

转换运算符将一个表达式的类型转换为另一个类型。

```rust
fn test(float x) {
    return 1 + u32(x);
}

test(3.14159); // 4
```
