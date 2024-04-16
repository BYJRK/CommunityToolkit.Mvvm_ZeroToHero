---
comments: true
---

# 中继指令（RelayCommand）

WPF 等框架中，如果希望为一个按钮控件绑定一个命令，正确的做法是为它的 `Command` 属性绑定一个 `ICommand` 接口类型的对象。但是，`ICommand` 接口的实现类通常需要自己实现 `CanExecute` 和 `Execute` 方法，这样会导致代码冗余。为了解决这个问题，于是有了“中继指令”（RelayCommand）这个概念。

!!! note
    “中继指令”这个概念并没有一个官方的名称。常见的名称有 `RelayCommand`、`DelegateCommand` 等。在 `ReactiveUI` 中，它的名称为 `ReactiveCommand`。
    
    目前主流的方式为使用一个类，并将实现使用方法委托传给这个类。但也有其他的方式，比如 SingletonSean 在他的 MVVM 相关的教学视频中为每个中继指令都单独写了一个实现。

`ICommand` 接口的定义如下：

```csharp
public interface ICommand
{
    bool CanExecute(object parameter);
    void Execute(object parameter);
    event EventHandler CanExecuteChanged;
}
```

但是工具包并不满足于此，而是又给出了几个额外的接口，包括：

- `IRelayCommand`
- `IRelayCommand<T>`
- `IAsyncRelayCommand`
- `IAsyncRelayCommand<T>`

其中，泛型版本为表示该中继指令可以传入一个参数，通常也就是按钮等控件的 `CommandParameter` 属性。

## `IRelayCommand` 接口

`IRelayCommand` 与 `IRelayCommmand<T>` 接口的定义分别为：

```csharp
public interface IRelayCommand : ICommand
{
    void NotifyCanExecuteChanged();
}

public interface IRelayCommand<in T> : IRelayCommand
{
    bool CanExecute(T? parameter);

    void Execute(T? parameter);
}
```

它在 `ICommand` 的基础上增加了一个 `NotifyCanExecuteChanged` 方法，用于通知命令的可执行状态发生了变化。这样，我们就可以更便捷地实现这一功能了。比如我们可以在某个属性的 `setter` 中调用这个方法，这样当属性值发生变化时，命令的可执行状态也会及时得到更新。

!!! info
    `IRelayCommand<in T>` 这里的 `in` 表示 [泛型逆变](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-generic-modifier)，这样我们就可以将一个 `IRelayCommand<object>` 赋值给一个 `IRelayCommand<string>` 类型的变量。

## `IAsyncRelayCommand`

这个接口的定义为：

```csharp
public interface IAsyncRelayCommand : IRelayCommand, INotifyPropertyChanged
{
    Task? ExecutionTask { get; }

    bool CanBeCanceled { get; }

    bool IsCancellationRequested { get; }

    bool IsRunning { get; }

    Task ExecuteAsync(object? parameter);

    void Cancel();
}
```

它在 `IRelayCommand` 的基础上增加了一些异步操作相关的属性和方法，比如 `ExecutionTask` 属性用于获取当前的执行任务，`Cancel` 方法用于取消当前的执行任务，`IsRunning` 属性用于判断当前异步任务是否正在执行等。

---

工具包当然不可能只为我们提供了一些接口，它还为我们提供了一些实现类，甚至还提供了非常好用的 [源生成器](../Source%20Generator/index.md)。关于这些内容，我们将在后续的文章中进行介绍。