---
comments: true
---

# 中继指令 RelayCommand

!!! warning
    虽然原生的 `ICommand` 接口以及 WPF 框架本身并没有对于中继指令的传参有任何要求，但是**工具包要求我们必须提供默认值（`default`）为 `null` 的参数**，否则按钮将会强制处于禁用状态！

    如果希望传参是一个整数，那么一定要写成 `int?`（等价于 `Nullable<int>`），表示一个可为空的整数。

工具包为我们提供了一个 `RelayCommand` 类，用于快捷地实现中继指令。它最朴素的用法是：

```csharp
public class ViewModel : ObservableObject
{
    public IRelayCommand MyCommand1 { get; } // (1)!
    public IRelayCommand MyCommand2 { get; }

    public ViewModel()
    {
        MyCommand1 = new RelayCommand(Execute, CanExecute); // (2)!
        MyCommand2 = new (() => {
            DoJob();
            Debug.WriteLine();
        }); // (3)!
    }

    private void Execute()
    {
        DoJob();
    }

    private bool CanExecute()
    {
        return true;
    }

    private void DoJob() { }
}
```

1. 通常我们会将类型声明为 `IRelayCommand`，而不是 `RelayCommand`，从而让使用者只关注接口暴露的方法，而不关心其实现；并且通常我们会在 VM 的构造函数中初始化这些属性，所以只提供 `getter` 即可
2. `RelayCommand` 的构造函数可以接受两个参数：`Execute` 和 `CanExecute`，分别表示执行和判断是否可执行的方法
3. `RelayCommand` 的构造函数也支持只传入一个参数，表示执行的方法，且不提供 `CanExecute` 的逻辑。而且这里除了直接传入符合委托类型的方法，还可以使用一个 Lambda 表达式简单地进行实现。

类似地，`RelayCommand<T>` 方法传入的是包含一个 `T` 类型入参的 `Execute` 与 `CanExecute` 方法。这里不再赘述。

## 借助源生成器来快速实现

但是在实际使用工具包进行开发时，我们基本不会采用上述朴素的方式进行实现，而是会借助源生成器。方式如下：

```csharp
public partial class ViewModel : ObservableObject
{
    [ObservableProperty]
    [NotifyCanExecuteChangedFor(nameof(SubmitCommand))]
    private string name;

    private bool CanSubmit() => !string.IsNullOrEmpty(name);

    [RelayCommand(CanExecute = nameof(CanSubmit))]
    private void Submit()
    {
        Debug.WriteLine($"Name: {name}");
    }
}
```

!!! tip
    如果 `CanExecute` 与一个 `bool` 类型的属性的值有关，那么特性中的 `CanExecute` 参数可以直接写为 `CanExecute = nameof(IsActive)`，而不需要另写一个方法去返回属性的值。

后台生成的代码形如

```csharp
partial class MainViewModel
{
    private RelayCommand? submitCommand;
    public IRelayCommand SubmitCommand
        => submitCommand ??= new RelayCommand(new Action(Submit), CanSubmit);
}
```

的属性。不难看出，这个属性的名称就是方法名后面加上 `Command`。

!!! info
    借助源生成器为方法后台生成的代码同样可以在项目的 `依赖项|分析器|CommunityToolkit.Mvvm.SourceGenerators` 中看到。可以参考 [相关章节](../Source%20Generator/FieldAttributes.md#_1) 的介绍。
