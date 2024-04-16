---
comments: true
---

# 异步中继指令 AsyncRelayCommand

异步中继指令更加强大，因为它背后可以是一个异步任务。这个异步任务的状态默认会对按钮控件的可用性进行控制，并且异步任务有办法取消。

我们跳过最朴素的用法，直接展示典型的使用源生成器的例子。

## 基本用法

```csharp
partial class MainViewModel : ObservableValidator
{
    [ObservableProperty]
    private string name = "";

    private bool CanSubmit() => !string.IsNullOrEmpty(Name);

    [RelayCommand(CanExecute = nameof(CanSubmit))]
    private async Task SubmitAsync()
    {
        await Task.Delay(1000);
    }
}
```

!!! note
    后台会为我们生成名为 `SubmitCommand` 的属性。工具包在生成这个属性时，会自动去掉原始方法名的 `Async` 结尾。

运行程序后会发现，在点击按钮后，按钮会自动灰掉 1 秒钟。这是因为期间异步任务正在执行，所以按钮会不可用。

## 取消任务

如果我们想要为这个异步中继指令添加取消功能，除了手动实现，我们还可以借助源生成器。只需要将上例修改为：

```csharp
[RelayCommand(CanExecute = nameof(CanSubmit), IncludeCancelCommand = true)]
private async Task SubmitAsync(CancellationToken token)
{
    try
    {
        await Task.Delay(2000, token);
    }
    catch (OperationCanceledException)
    {

    }
}
```

即可。此时后台会为我们额外生成一个名为 `SubmitCancelCommand` 的中继指令。此时只需要将该中继指令绑定到一个按钮，即可使用。并且运行程序可以发现，这个取消按钮只有在异步任务执行时才会处于可用状态。

此时的 XAML 内容形如：

```xml
<StackPanel>
    <TextBox Text="{Binding Name"} />
    <Button Content="Submit" Command="{Binding SubmitCommand}" />
    <Button Content="Cancel" Command="{Binding SubmitCancelCommand}" />
</StackPanel>
```

!!! note
    如果希望异步任务可以被取消，此时方法必须接受一个 `CancellationToken` 类型的参数。如果此时还希望该方法能接受一个入参，那么需要将入参作为第一个参数，`CancellationToken` 作为第二个参数。

## 异步任务可以多次发起

如果我们希望异步任务可以多次发起，也就是按钮不需要等待上一个任务结束才能继续点击，那么只需要将特性的 `AllowConcurrentExecutions` 参数设为 `true` 即可。