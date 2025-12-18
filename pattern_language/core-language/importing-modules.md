# Importing Modules

模式可拆分为多个文件，以便将模式代码更合理地划分为逻辑部分。为将其他文件导入当前文件，模式语言提供了两种机制：`#include` 指令和 `import` 语句。两者均会在模式搜索路径中的 `includes` 文件夹内查找指定文件。默认搜索路径如下：

* ImHex 安装目录。
* `AppData/Local/imhex` 目录。
* 附加搜索路径。要添加附加搜索路径，请转至 `工具 > 设置 > 文件夹` 菜单。

### `#include` directive

A [preprocessor directive](preprocessor.md). 预处理器将此指令替换为包含文件中的所有词法标记——包括所有预处理器定义。在包含时，该系统会维护自己的文件列表，这些文件标记为  [`#pragma once`](importing-modules.md#include-guards).

要使用 `#include` 指令，请指定要包含的文件路径并跟上文件扩展名。路径可以使用双引号（`“path/filename.extension”`）或尖括号（`<path/filename.extension>`）包裹。扩展名可选。若路径未指定扩展名，预处理器将在指定路径的`pattern`文件夹中查找文件，例如`path/pattern/filename`。当未指定扩展名时，`#include`指令会搜索扩展名为`.pat`和`.hexpat`的文件。但按惯例，包含文件时应明确指定扩展名。以下两种`#include`指令用法均有效：

```rust
#include "std/io.pat"
#include <std/string.pat>
```

### `import` statement

`import`语句在解析阶段进行处理。解析器遇到该语句后，会创建独立的解析器将导入文件作为单独的编译单元进行解析，随后将其插入当前文件的抽象语法树（AST）。预处理器在使用`import`语句时定义为_不传播_。

`import` 关键字后需跟随待包含文件的路径，使用点号（`.`）作为目录分隔符。文件扩展名将自动解析。该语句会搜索扩展名为 `.pat` 和 `.hexpat` 的文件。与其他语言语句相同，该行末尾必须以分号（`;`）结束。

```rust
import sys.mem;
```

`import` 语句也可以接受字符串参数。这种用法与 [`#include`](importing-modules.md#include-directive) 指令行为一致。当模块名称或文件夹包含关键字时可使用此方法，因为关键字不能用作标识符。

```rust
// Since match is a keyword, we can't import a module named "match.pat" as described
// in the previous example. We can pass a string argument to import instead.
import "tools/match.pat";
```

### `import as` statement

库通常会将所有内容置于特定命名空间下，以避免不同库之间的名称冲突。然而这往往导致代码变得冗长。

为解决此问题，可通过`as`关键字更改类型和函数导入时的命名空间。

```cpp
import std.ctype;

char character @ 0x00;
if (std::ctype::isdigit(character)) {
    // ...
}


// --------------

import std.ctype as c;

char character @ 0x00;
if (c::isdigit(character)) {
    // ...
}
```

### Importing other Patterns

在处理文件格式时，有时会遇到其他文件或格式嵌套在当前数据中的情况。例如，归档文件中可能包含头部信息和文件表，随后是其他格式的数据——比如归档文件中可能并存可执行文件和图像文件。

在这种情况下，通常需要将其他模式导入当前模式，以便这些其他格式的代码仍可独立使用。

为此，可使用 `import * from <模式文件> as <类型名称>` 语句实现。

```python
import * from pe as Executable;

Executable executable @ 0x1234;
```

此代码根据`pe.hexpat`模式文件的内容创建名为`Executable`的新结构体类型。当前模式中任何位置放置`Executable`类型时，都会评估导入模式中的代码——因为内容已被复制粘贴到当前文件中，且所有偏移量均已调整。

以下实现细节值得关注：

* 当将创建的类型放置在地址`0x1000`时，导入的模式会将文件视为从`0x1000`地址开始。它无法访问`0x1000`之前的任何字节，并将文件中的`0x1000`地址视为`0x00`地址。
* 若导入模式文件中仅存在单个全局放置，该模式内容将被内联到新创建的结构体中，避免产生额外间接引用。若存在多个全局放置，则分别独立放置于创建的结构体中。

### `auto namespace`

由于单个库文件可能包含多个命名空间，且并非总是希望将所有命名空间的内容都导入到同一个重命名后的命名空间中，因此可以指定需要导入并重命名的具体命名空间。

要实现这一点，请将该命名空间标记为 `auto`：

```cpp
// std/ctype.pat

namespace auto std::ctype {

    // ...

}

namespace impl {

    // ...

}
```

```cpp
// Import the std::type namespace from the ctype.pat library file
// and rename the std::ctype namespace to test
import std.ctype as test;
```

### Include guards

#### `#pragma once` directive

为避免重复声明，用于导入其他文件的文件可使用`#pragma once`指令防止多次包含。**重要提示**：`#include`指令和`import`语句各自维护着标记为`#pragma once`的文件列表。这意味着当某个文件被当前文件包含后，又通过导入其他文件被间接包含时，当前文件将出现重复声明。换言之，为避免重复声明，文件应仅通过以下任一机制被包含：`#include`指令或`import`语句。

#### Manual include guards

使用 `#include` 时，可通过 `#ifndef` 和 `#define` 指令手动编写包含保护：

```c++
#ifndef FOO_HEXPAT
#define FOO_HEXPAT

// Pattern code

#endif
```

#### Importing standard library headers

标准库头文件内部均使用`import`语句，因此应通过`import`进行导入。

```rust
import std.io;
import std.mem;
import type.float16;
```

_注意_，这仅适用于标准库头文件。自定义模式仍可导入标准库头文件，同时使用`#include`包含客户端项目中的文件。

### Forward declarations.

The pattern language supports [forward declarations](data-types.md#forward-declaration).
