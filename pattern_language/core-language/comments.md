# 注释

模式语言与大多数编程语言一样，支持注释功能。注释是指被解释器忽略的文本片段。注释既可用于记录代码说明，也可用于临时禁用代码。

### Single line comments

单行注释以双斜杠 (//) 开头，并延续至行尾。

```cpp
// This is a single line comment
```

### Multi line comments

多行注释以 /\* 开头，以 \*/ 结束。

```cpp
/* This is
    a multi
    line comment */
```

### 文档注释

文档注释用于为整个模式、单个函数或类型提供额外的文档说明。

编写文档注释有多种方式：

```rust
/*!
    This is a global doc comment.
    It documents the whole pattern and can contain various attributes that can be used by tools to extract information about the pattern.
*/

/**
    This is a local doc comment.
    It documents the function or type that immediately follows it.
*/

/**
    This is a doc comment documenting a function that adds two numbers together
    @param x The first parameter.
    @param y The second parameter.
    @return The sum of the two parameters.
*/
fn add(u32 x, u32 y) {
    return x + y;
};

/// This is a single line local comment. It documents the function or type that immediately follows it.
```
