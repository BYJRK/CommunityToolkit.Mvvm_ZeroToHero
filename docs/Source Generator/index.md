# 源生成器（Source Generator）

[:material-file-document: 官方文档](https://learn.microsoft.com/zh-cn/dotnet/communitytoolkit/mvvm/generators/overview){.md-button }

为了让我们在书写 ViewModel 时能够更加便捷高效，工具包为我们提供了一个非常强大的功能：源生成器。借助这一功能，我们可以非常轻易地写出功能完整的属性（Property）及中继指令（RelayCommand）。

下面是一个简单的示例：

```csharp
public partial class MyViewModel : ObservableObject
{
    [ObservableProperty]
    private string name = "Tom";

    [RelayCommand]
    private void Foo()
    {
        Name = "Jerry";
    }
}
```

使用源生成器的方式非常简单，只需要：

1. 将继承了 `ObservableObject` 的类添加 `partial` 关键字（这是源生成器的要求，因为需要通过分部类的方式在后台生成部分代码）
2. 在字段和方法上添加一些特性

然后工具包就会在后台生成相应的代码。以上面的代码为例，后台大概会生成这样的内容（节选并简化了部分代码）：

```csharp
public partial class MyViewModel : ObservableObject
{
    /// <inheritdoc cref="name"/>
    public string Name
    {
        get => name;
        set
        {
            if (!EqualityComparer<string>.Default.Equals(name, value))
            {
                OnNameChanging(value);
                OnNameChanging(default, value);
                OnPropertyChanging("Name");
                name = value;
                OnNameChanged(value);
                OnNameChanged(default, value);
                OnPropertyChanged("Name");
            }
        }
    }

    partial void OnNameChanging(string value);
    partial void OnNameChanging(string? oldValue, string newValue);
    partial void OnNameChanged(string value);
    partial void OnNameChanged(string? oldValue, string newValue);

    /// <summary>The backing field for <see cref="FooCommand"/>.</summary>
    private RelayCommand? fooCommand;
    public IRelayCommand FooCommand => fooCommand ??= new RelayCommand(new Action(Foo));
}
```

这样一来，我们就可以非常轻松地书写 ViewModel 了。当然，仅借助上面例子中给出的特性是远远不够的，因为在实际开发中，我们经常还要面临这样的情形：

- `FullName` 希望在 `FirstName` 发生变化时也得到通知
- `FirstName` 属性在发生变化后希望调用一个方法，执行额外的逻辑
- `Foo` 方法希望接收一个参数
- `Foo` 方法的 `CanExecute` 与某个属性或方法的返回值相关联
- `Foo` 是一个异步方法，且可以被取消
- ……

这时候就需要借助更多的特性来完成这些需求。在下面的章节中，我们会详细介绍工具包为我们提供的全部与源生成器相关的功能与特性。
