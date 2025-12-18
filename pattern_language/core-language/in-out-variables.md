# In / Out Variables

输入和输出变量是将配置或输入数据传递至模式，并从模式中读取结果数据的一种方式。

每个`in`和`out`变量都会在模式编辑器视图的“设置”选项卡中创建一个条目。`in`变量创建输入字段，`out`变量创建标签。

在执行模式之前，用户可在`in`变量的输入字段中输入值。该值将在模式执行前被复制到对应的`in`变量中。

同样地，模式执行完毕后，写入`out`变量的任何值都将显示在标签中。

以下代码展示了一个简单模式：读取`inputValue`的值，将其乘以二，然后写回`outputValue`。

```rust
u32 inputValue in;
u32 outputValue out;

fn main() {
    outputValue = inputValue * 2;
};
```

<figure><img src="../.gitbook/assets/in_out/settings.png" alt=""><figcaption></figcaption></figure>
