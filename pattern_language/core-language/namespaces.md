---
description: Isolating functions and types
---

# Namespaces

命名空间提供了一种封装和分组多个相似类型的机制。它们还允许不同命名空间中的多个类型使用相同名称而互不干扰。

```cpp
namespace abc {

    struct Type {
        u32 x;
    };

}

abc::Type type1 @ 0x100;

using Type = abc::Type;
Type type2 @ 0x200;
```

要访问命名空间内的类型，需使用作用域解析运算符 `::`。在上例中，要访问命名空间 `abc` 内的类型 `Type`，应使用 `abc::Type`。

## Nested and Reopened Namespaces

您还可以定义**嵌套命名空间**并在后续重新打开它们。这对于组织与主类型相关的内部常量和辅助逻辑尤为有用，同时能保持它们的封装性。

```cpp
namespace game {

    struct User {
        u32 id;
        u32 age;
    } [[format("game::impl::format_user")]];

    namespace impl {
        fn format_user(User user) {
            return std::format("User({}, {})", user.id, user.age);
        };
    }

    struct Score {
        u32 points;
        u32 level;
    } [[format("game::impl::format_score")]];

    namespace impl {
        fn format_score(Score score) {
            return std::format("Score({}, {})", score.points, score.level);
        };
    }

}
```

These separate blocks both add to the same `game::impl` namespace. This provides a clean separation between public type definitions and internal helper logic and improves readablity.
