---
comments: true
---

# TaskNotifier

在 MVVM 模式中，我们有时需要将一个异步操作作为属性暴露给 UI 绑定（例如显示加载状态、展示任务结果或捕获异常）。

如果我们直接给视图模型添加一个 `Task` 类型的属性（也就是使用普通的 `SetProperty`），那么它只会在给属性赋予新对象时触发一次 `PropertyChanged` 事件。此时任务通常才刚刚启动，UI 绑定的还是初始/未完成状态；当任务真正执行完毕时，由于属性指向的 Task 对象引用并没有改变，也没有什么机制去触发通知，会导致 UI 无法自动更新为完成后的结果。

为了解决这个问题，`ObservableObject` 提供了 `SetPropertyAndNotifyOnCompletion` 方法，并配合 `TaskNotifier` / `TaskNotifier<T>` 类型来管理异步属性的通知。

## 工作机制

`SetPropertyAndNotifyOnCompletion` 会在以下两个时机分别触发属性通知：

1. **赋值时**：当为属性赋予一个新的 `Task` 实例时，先触发一次 `PropertyChanged`，通知 UI 任务已经开始（此时 `Task.IsCompleted` 为 `false`）。
2. **任务完成时**：在后台监控该任务，当任务执行完毕（无论是成功、取消还是抛出异常）后，会自动**再次触发** `PropertyChanged` 事件，通知 UI 刷新绑定。

为了能够追踪并管理任务状态，该方法要求后备字段使用特殊的包装类型 `TaskNotifier`（对应无返回值的 `Task`）或 `TaskNotifier<T>`（对应带返回值的 `Task<T>`）。

!!! note "隐式类型转换"
    `TaskNotifier` 和 `TaskNotifier<T>` 是 `ObservableObject` 的受保护（`protected`）内部类。它们内部定义了到 `Task` / `Task<T>` 的**隐式转换运算符**（`implicit operator`），因此在属性的 `getter` 中可以直接作为普通的 `Task` 返回，无需手动解包。

## 基本用法

### 1. 返回结果的异步任务 (`Task<T>`)

当异步操作有返回值时，后备字段声明为 `TaskNotifier<T>`，公开属性声明为 `Task<T>`：

```csharp
public class MyViewModel : ObservableObject
{
    // 1. 后备字段使用 TaskNotifier<T>
    private TaskNotifier<string>? _loadDataTask;

    // 2. 属性类型依然是 Task<T>，在 setter 中调用 SetPropertyAndNotifyOnCompletion
    public Task<string>? LoadDataTask
    {
        get => _loadDataTask;
        set => SetPropertyAndNotifyOnCompletion(ref _loadDataTask, value);
    }

    public void LoadData()
    {
        // 3. 直接将 Task<T> 赋值给属性即可
        LoadDataTask = FetchDataAsync();
    }

    private async Task<string> FetchDataAsync()
    {
        await Task.Delay(2000);
        return "数据加载成功！";
    }
}
```

### 2. 无返回值的异步任务 (`Task`)

当异步操作没有返回值时，使用非泛型的 `TaskNotifier` 与 `Task`：

```csharp
public class MyViewModel : ObservableObject
{
    private TaskNotifier? _saveTask;

    public Task? SaveTask
    {
        get => _saveTask;
        set => SetPropertyAndNotifyOnCompletion(ref _saveTask, value);
    }

    public void Save()
    {
        SaveTask = DoSaveAsync();
    }

    private async Task DoSaveAsync()
    {
        await Task.Delay(1000);
    }
}
```

## 任务完成回调

`SetPropertyAndNotifyOnCompletion` 还提供了支持回调函数（`Action<Task>` 或 `Action<Task<T>>`）的重载版本。当任务完成触发属性通知后，会自动执行指定的回调：

```csharp
public class MyViewModel : ObservableObject
{
    private TaskNotifier<string>? _loadDataTask;

    public Task<string>? LoadDataTask
    {
        get => _loadDataTask;
        set => SetPropertyAndNotifyOnCompletion(ref _loadDataTask, value, task =>
        {
            if (task.IsCompletedSuccessfully)
            {
                // 任务成功完成后的处理逻辑
                Debug.WriteLine($"加载完成: {task.Result}");
            }
            else if (task.IsFaulted)
            {
                // 处理异常
                Debug.WriteLine($"加载失败: {task.Exception?.Message}");
            }
        });
    }
}
```

## XAML 界面绑定

因为公开属性是标准的 `Task` / `Task<T>` 实例，因此可以直接在 XAML 中绑定 `Task` 类的原生属性：

```xml
<StackPanel>
    <!-- 绑定任务执行结果（完成前为 null/默认值，完成后自动刷新显示） -->
    <TextBlock Text="{Binding LoadDataTask.Result}" />

    <!-- 绑定任务是否已完成（可配合转换器控制加载指示器的显示/隐藏） -->
    <ProgressBar IsIndeterminate="True"
                 Visibility="{Binding LoadDataTask.IsCompleted, Converter={StaticResource BooleanToInverseVisibilityConverter}}" />

    <!-- 绑定任务抛出的异常信息 -->
    <TextBlock Text="{Binding LoadDataTask.Exception.InnerException.Message}"
               Foreground="Red" />
</StackPanel>
```

## 与 AsyncRelayCommand 的对比与选型

如果你的需求主要是**响应用户操作触发异步任务**，并且想要**获取任务是否正在运行**（例如在界面上展示 Loading 加载状态或防止按钮被重复点击），还可以考虑使用 [AsyncRelayCommand](../../RelayCommand/AsyncRelayCommand.md)（或源生成器的 `[RelayCommand]` 特性）：

- **`AsyncRelayCommand`**：自带 `IsRunning` 属性与任务取消支持，执行期间会自动禁用绑定的按钮（防止重复触发），配合源生成器无需手写样板代码。
- **`TaskNotifier`**：适用于**数据驱动**的场景，主要用于需要在 ViewModel 中直接将 `Task` / `Task<T>` 作为属性公开给 UI 绑定并监听其完成状态（例如直接在 XAML 中绑定 `Task.Result` 或捕获任务异常）。

## 在 Avalonia UI 中的使用差异

Avalonia 与 WPF 存在一些区别，导致它不需要也不建议使用 `TaskNotifier`**：

1. **通知机制差异**：Avalonia 的数据绑定底层机制与 WPF 不同。`TaskNotifier` 在任务完成后，针对同一个 `Task` 实例再次触发 `PropertyChanged` 事件，无法促使 Avalonia 的绑定系统正确重新读取并刷新属性值。
2. **UI 阻塞风险**：在 Avalonia（尤其是启用编译绑定时）直接在 XAML 中通过 `Task.Result` 访问未完成的任务，可能会导致 UI 线程阻塞。

### Avalonia 推荐方案：`^` 流绑定运算符

Avalonia 框架本身原生内置了**异步流式绑定运算符 `^`**（Stream Binding）。当绑定到 `Task` 或 `Task<T>` 类型的属性时，只需在属性名后追加 `^`，Avalonia 就会自动异步等待任务完成并更新界面，期间还可以通过 `FallbackValue` 指定加载阶段的占位内容。

因此，在 Avalonia 中直接使用普通属性即可：

**ViewModel 代码：**

```csharp
public partial class MyViewModel : ObservableObject
{
    // 在 Avalonia 中直接使用普通的 Task<T> 属性，无需 TaskNotifier
    [ObservableProperty]
    private Task<string>? _loadDataTask;

    public void LoadData()
    {
        LoadDataTask = FetchDataAsync();
    }

    private async Task<string> FetchDataAsync()
    {
        await Task.Delay(2000);
        return "数据加载成功！";
    }
}
```

**Avalonia AXAML 绑定：**

```xml
<StackPanel>
    <TextBlock Text="{Binding LoadDataTask^, FallbackValue='数据加载中...'}" />
</StackPanel>
```
