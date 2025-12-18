# Preprocessor

预处理器的运作方式与C/C++中的类似。所有以`#`符号开头的行都被视为预处理指令，并在分析程序其余部分的语法之前进行评估。

### `#define`

```cpp
#define MY_CONSTANT 1337
```

该指令将执行查找替换操作。在上例中，标签`MY_CONSTANT`将在整个程序中被替换为`1337`，且不进行任何词法分析。这意味着即使在字符串内部，该指令也会被替换。此外，若使用多个定义，后续的查找替换操作可能会修改先前操作已改变的表达式。
### `#include`

```cpp
#include <mylibrary.hexpat>
```

此指令允许将其他文件包含到当前程序中。指定文件的内容将直接复制到当前文件中。更多信息请参阅 [importing modules](importing-modules.md#include-directive) .

### `#ifdef`, `#ifndef`, `#endif`

```cpp
#ifdef SOME_DEFINE
    u32 value @ 0x00;
#endif
```

这些预处理指令可检查给定定义是否已被定义。`#ifdef` 在给定定义存在时包含其与闭合指令 `#endif` 之间的所有内容。`#ifndef` 与 `#ifdef` 功能相同，但在给定定义不存在时包含其所有内容。
### `#error`

```cpp
#error "Something went wrong!"
```

若在预处理阶段遇到该指令，将抛出错误。该功能主要配合`#ifdef`和`#ifndef`使用，用于检查特定条件。

### `#pragma`

```cpp
#pragma endian big
```

编译指示是给运行时的提示，用于告知其如何处理某些事项。

可用的编译指示如下：

#### `endian`

**Possible values:** `big`, `little`, `native` **Default:** `native`

此指令覆盖文件中所有变量声明的默认字节序。

#### `MIME`

**Possible values:** Any MIME Type string **Default:** `Unspecified`

此指令用于指定可由该模式解析的文件的MIME类型。当文件被打开时，此功能可自动加载相关模式。系统会将加载文件的MIME类型与此处指定的类型进行比对，若匹配成功，则弹出提示框询问是否加载该模式。

#### `magic`

**Possible values:** A byte pattern in the form of `[ AA BB ?? D? ] @ 0x00`

此指令用于指定二进制模式，该模式用于检查加载的数据，以确定该模式是否可用于解析此数据。

该模式包含两部分：第一部分是十六进制值列表，其中?表示通配符。该列表按顺序与数据进行比对，标记为通配符的半字节将被忽略且不参与比较。第二部分是@符号后面的十六进制值，该值被解释为在数据中查找该模式的起始地址。

#### `base_address`

**Possible values:** Any integer value **Default:** `0x00`

此指令会自动调整当前加载文件的基址。对于依赖文件在内存中特定地址加载的模式而言，此功能非常有用。


#### `eval_depth`

**Possible values:** Any integer value **Default:** `32`

此指令用于设定递归函数和类型的求值深度。为防止运行时在评估无限深递归类型时崩溃，若检测到递归过深，执行将提前终止。此指令可调整允许的最大深度。

#### `array_limit`

**Possible values:** Any integer value **Default:** `0x1000`

此指令用于设定数组允许的最大条目数量。为防止运行时创建超大数组时消耗过多内存，若评估的数组条目过多，程序将提前终止执行。该指令可调整允许的最大条目数量。

#### `pattern_limit`

**Possible values:** Any integer value **Default:** `0x2000`

此指令用于设定允许创建的模式最大数量。为防止运行时在创建大量模式时耗尽内存，当同时存在的模式数量过多时，程序将提前终止执行。该指令与`array_limit`指令功能相似，但同时也能捕获较小的嵌套数组。

#### `once`

此指令不接受任何参数，仅标记当前文件仅可被包含一次。这意味着若文件被多次包含（例如先被显式包含，随后又在另一个包含文件中再次被包含），则仅执行首次包含操作。

此特性主要用于防止文件中定义的函数、类型和变量被重复定义。

`import`语句与`#include`指令各自维护着独立的`#pragma once`标记文件列表。因此采用统一导入机制的头文件集应保持一致性。更多信息请参阅[Importing Modules](importing-modules.md) .

#### `bitfield_order`

**Possible values:** `right_to_left`, `left_to_right` **Default:** `right_to_left`

此指令覆盖默认位域的位序。其作用与`[[left_to_right]]`和`[[right_to_left]]`属性相同，但会自动应用于所有创建的位域。

#### `debug`

此指令用于在评估器中启用调试模式，将触发以下行为：

* 任何作用域的压入与弹出操作均会记录至控制台
* 任何内存访问操作均会记录至控制台
* 任何变量的创建与赋值操作均会记录至控制台
* 任何函数调用及其参数均会记录至控制台
* 若发生错误，已存入内存的模式将不会被删除

#### `author`

**Possible values:** Any string value **Default:** `Unspecified`

此指令用于指定[Content Store](../../imhex/misc/content-store.md).中显示的模式作者。若需指定多位作者，仅使用一个`#pragma author`指令。

#### `description`

**Possible values:** Any string value **Default:** `Unspecified`

此指令用于指定在模式中显示的描述。 [Content Store](../../imhex/misc/content-store.md).
