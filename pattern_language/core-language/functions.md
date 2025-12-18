# Functions

函数是可复用的代码片段，能够执行计算。它们与其他编程语言中的函数基本相同。

参数类型需要显式指定，返回类型则会自动推断。

```rust
fn min(s32 a, s32 b) {
    if (a > b)
        return b;
    else
        return a;
};

std::print(min(100, 200)); // 100
```

要接受数组参数，请将其类型指定为 `ref auto`。

```rust
fn print_array(ref auto arr) {
    std::print("{}", arr);
    for (auto i = 0, i < std::core::member_count(arr), i += 1) {
        std::print("{}: {}", i, arr[i]);
    }
};
u8 v[1];
v[0] = 1;
print_array(v);  // 0: 1
```

### Global Scope

全局作用域在很大程度上与函数类似。程序执行时，全局作用域中的所有语句都会被执行。唯一的区别在于，全局作用域中可以定义新的类型。

### Parameter packs

为允许向函数传递无限数量的参数，可使用参数包。

```rust
fn print_sequence(auto first, auto ... rest) {
    std::print("{}", first);

    if (std::sizeof_pack(rest) > 0)
        print_sequence(rest);
};

print_sequence(1, 2, 3, 4, 5, 6);
```

参数包只能作为其他函数的参数使用。使用时会自动展开参数包，其作用如同将包含的值逐个传递给其他函数。

上述函数将按顺序打印所有传递的值：先输出第一个参数，然后在下次运行时不将其传递给函数，从而移除该参数。

### Default parameters

默认参数可用于为参数设置默认值，当函数被调用时若未提供参数即可生效。

```rust
fn print_numbers(u32 a, u32 b, u32 c = 3, u32 d = 4) {
    std::print("{} {} {} {}", a, b, c, d);
};

print_numbers(1, 2);            // Prints 1 2 3 4
print_numbers(8, 9, 10, 11);    // Prints 8 9 10 11
```

### Reference parameters

引用参数可用于避免不必要地复制大量数据，在需要向函数传递没有固定布局的自定义类型时尤为有用。它不会将给定的参数值复制到函数的堆上，而是使用对现有模式的引用。

```rust
struct MyString {
    char value[while(std::mem::read_unsigned($, 1) != 0xFF)];
};

fn print_my_string(ref MyString myString) {
    std::print(myString.value);
};
```

### Variables

变量可以在函数外部以类似的方式声明，但它们会被放置在函数的栈上，而不是被高亮显示。

```rust
fn get_value() {
    u32 value;
    u8 x = 1234;

    value = x * 2;

    return value;
};
```

自定义类型也可在函数内部使用

```rust
union FloatConverter {
    u32 integer;
    float floatingPoint;
};

fn interpret_as_float(u32 integer) {
    FloatConverter converter;

    converter.integer = integer;
    return converter.floatingPoint;
};
```

也可以使用`const`关键字声明常量

```rust
fn get_value() {
    const u32 value = 1234;
    return value;
};

std::print("{}", get_value()); // 1234
```

### Control statements

#### If-Else-Statements

if、else-if 和 else 语句的工作方式与大多数其他 C 类语言相同。当 `if` 语句头中的条件评估为真时，其主体中的代码将被执行。若评估为假，则执行可选的 `else` 代码块。

花括号是可选的，仅当主体中包含多条语句时才需要使用。

```rust
if (x > 5) {
    // Execute when x is greater than 5
} else if (x == 2) {
    // Execute only when x is equals to 2
} else {
    // Execute otherwise
}
```

#### While-Loops

while循环的工作原理与if语句类似。只要循环头的条件评估为真，循环体就会持续执行。

```rust
while (check()) {
    // Keeps on executing as long as the check() function returns true
}
```

#### For-Loops

for循环是另一种与while循环相似的循环结构。其头部由三个用逗号分隔的代码块组成（例如`i < 10`）。

第一块用于声明迭代变量。该变量仅能在 for 循环内部使用，其初始值即为循环起始迭代次数。示例中第一块为 `u8 i = 0`。

第二块是用于判断循环何时继续运行、何时终止的条件。每次循环迭代时都会检查此条件。示例中第二块为`i < 10`。每次循环开始时，系统会检查`i`当前值是否仍小于`10`。当`i`达到`10`或更大时，循环将停止执行。只要该块中的语句评估为`true`，循环就会持续进行。

第三个代码块仅在循环主体执行后运行。它为迭代变量赋予新数值——通过指定增量进行递增。示例中采用`1`作为增量，但增量值并非固定。若需递增`2`、`3`等数值，只需将`1`替换为相应数字即可。

```rust
// Declare a variable called i available only inside the for
for (u8 i = 0, i < 10, i = i + 1) {
    // Keeps on executing as long as i is less than 10

    // At the end, increment i by 1
}
```

赋值运算符 `+=` 也可用于递增 for 循环：

```rust
for (u8 i = 0, i < 10, i += 1) {
    // Body of loop
}
```

### Loop control flow statements

在循环内部，可使用`break`和`continue`关键字控制循环内的执行流程。

当遇到`break`语句时，循环立即终止，代码流程跳转至循环外部继续执行。当遇到`continue`语句时，当前迭代立即终止，代码流程返回循环开头重新执行，并再次检查循环条件。

### Return statements

要从函数中返回值，需使用`return`关键字。

函数的返回类型将根据返回值自动确定。

```rust
fn get_value() {
    return 1234;
};

std::print("{}", get_value()); // 1234
```

### Pattern views

在全局作用域内的函数或函数语句（如`if`、`for`或`while`语句）中使用放置语法时，系统会创建该数据的视图。

视图的行为与放置模式极为相似——您可以像访问和传递放置变量那样操作该值。但与常规放置不同，它不会生成输出模式。

```rust
fn read_u32(u32 address) {
    u32 value @ address;

    return value;
};

std::print("{}", read_u32(0x1234)); // Prints the value at address 0x1234 formatted as a u32
```

### User-defined Literals

用户定义的字面量本质上是函数调用的语法糖，在某些情况下比常规函数调用更易于阅读。要创建此类字面量，只需定义一个以下划线开头且仅接受单个字符作为参数的函数即可。

```rust
fn _literal(u32 value) {
    return value * 2;
};

u32 two_times = 123_literal; // two_times = 246
```

同样可以定义接受多个参数的用户自定义字面量。此时，字面量所作用的值作为第一个参数传递，其余参数则作为第二个及后续参数传递。

```rust
fn _literal(u32 value, u32 multiplier) {
    return value * multiplier;
};

u32 three_times = 123_literal(3); // three_times = 369
```

任何内置类型均可作为第一个参数使用。这使得用户定义的字面量能够与整数、浮点数、字符字面量和字符串配合使用。
