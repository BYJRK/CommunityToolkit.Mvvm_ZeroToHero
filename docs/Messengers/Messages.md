---
comments: true
---

# 常用消息类型 Messages

工具包为我们提供了几个常用的消息类型，包括：

- `ValueChangedMessage`
- `PropertyChangedMessage`
- `RequestMessage`

这几种消息类型有的拥有一些特殊的功能，下面将会详细介绍。如果我们希望对消息类型进行定制或扩展，那么可以继承这些消息类型，并实现自己的消息类型。

## 自定义 Message

工具包中的 `IMessenger` 接口约定的 `Send`、`Register` 等泛型方法，并没有要求消息的种类满足任何条件（比如继承自某个基类），所以完全可以自由地实现任何消息类型。

比如这里，我们用 C# 9.0 为我们带来的 `record` 类型快速声明一个消息类型，并使用：

```csharp
// 声明一个消息类型
public record MyMessage(string Content);

// 发送消息
WeakReferenceMessenger.Default.Send(new MyMessage("Hello World!"));

// 接收消息
WeakReferenceMessenger.Default.Register<MyMessage>(this, message =>
{
    Console.WriteLine(message.Content);
});
```

## ValueChangedMessage

`ValueChangedMessage` 是一个最基本的消息类型，它包含一个 `Value` 属性，用于存储消息的值。如果希望发送一个用于通知某个值发生变化的消息，可以使用 `ValueChangedMessage`。

此外，还可以继承这个类，从而实现自己的消息类型。比如：

```csharp
public class MyMessage : ValueChangedMessage<string>
{
    public MyMessage(string value) : base(value)
    {
    }
}

// 发送消息
WeakReferenceMessenger.Default.Send(new MyMessage("Hello World!"));

// 接收消息
WeakReferenceMessenger.Default.Register<MyMessage>(this, message =>
{
    Console.WriteLine(message.Value);
});
```

!!! info "没有接收者？"
    对于一个不需要回复的消息（不是 `RequestMessage` 或其子类），如果没有任何接收者，那么 `Send` 方法将会正常运行，且不会有任何效果。

## RequestMessage

这是一个特殊的消息类型。如果 `Send` 方法发送的是这个类（或它的子类），那么 `Send` 方法将拥有一个返回值，这个返回值就是消息的接收者回复的消息。此时，消息的接收者也能够通过消息上的 `Reply` 方法回复消息的发送者。比如：

```csharp
// 注册消息接收者
WeakReferenceMessenger.Default.Register<MyRequestMessage>(
    new object(),
    (_, m) =>
    {
        // 收到消息后，会进行简单的判断，并回复消息
        if (m.Content == "Nice to meet you!")
            m.Reply("Nice to meet you, too!");
        else
            m.Reply("Yes?");
    }
);

// 发送消息并查看回复的消息内容
var res1 = WeakReferenceMessenger.Default.Send(new MyRequestMessage("Hello!"));
Console.WriteLine(res1.Response);

var res2 = WeakReferenceMessenger.Default.Send(new MyRequestMessage("Nice to meet you!"));
Console.WriteLine(res2.Response);

// 声明一个自定义消息类型
public class MyRequestMessage : RequestMessage<string>
{
    public string Content { get; init; }

    public MyRequestMessage(string content)
    {
        Content = content;
    }
}
```

然后就能在控制台看到这样的输出内容：

```
Yes?
Nice to meet you, too!
```

!!! warning "没有回复者？"
    由于在发送 `RequestMessage` 时，消息的接收者需要等待回复消息，所以如果此时没有接受者，那么可能会出现问题。具体来说，单纯调用 `Send` 不会报错，但是如果试图访问 `response.Response`，**则会抛出异常**。原因是并没有任何接收者提供了回复。

!!! warning "有多个回复者？"
    **如果有多个接收者回复了消息，那么将会报错**。此时正确的做法是，可以从 `RequestMessage` 上的 `HasReceivedResponse` 属性判断是否已经有接收者回复了消息。如果已经有接收者回复了消息，那么这个属性的值将会为 `true`。

## AsyncRequestMessage

`AsyncRequestMessage` 是 `RequestMessage` 的异步版本。但是它可能稍微有一点反常识。在使用它时，与其说是接收方在异步地处理消息并返回结果，不如说它直接将一个异步任务丢给了发送者，并让发送者自己在接到异步任务后开始等待任务的完成，并最终拿到结果。

!!! info "为什么要这样设计？"
    这样设计其实是有原因的：`IMessenger` 的 `Register` 方法中传入的回调是一个 `void` 类型的，也就是说如果我们想在接收者这边进行异步处理，我们只能给它传入一个 `async void` 的回调。这是很不理想的方式。

我们看一个简单的例子：

```csharp
// 接收方
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        WeakReferenceMessenger.Default.Register<AsyncRequestMessage<string>>(this, (_, m) =>
        {
            // 这里我们回复消息时，其实异步任务只是刚刚开始，还没有完成
            m.Reply(GetStringAsync());
        });
    }

    // 模拟一个需要耗时一段时间才能得到结果的任务
    private async Task<string> GetStringAsync()
    {
        await Task.Delay(2000);
        return "hello, world!";
    }
}

// 发送方
partial class MainViewModel : ObservableObject
{
    [RelayCommand]
    private async Task SendMessageAsync()
    {
        var request = WeakReferenceMessenger.Default.Send<AsyncRequestMessage<string>>();
        var response = await request.Response; // 这里的 Response 属性是一个 Task<string>
    }
}
```

通过这样的方式，我们就可以实现异步地接收消息回复了。

## CollectionRequestMessage

如果不满足于 `RequestMessage` 只能有一个接收者进行回复这一限制，而是希望多位接收者都能进行回复，并且回复的内容会被放在一个集合里面，那么 `CollectionRequestMessage<T>` 就派上用场了。这个消息类型实现了 `IEnumerable<T>` 接口，所以我们可以像使用集合一样使用它。此外，也可以访问它的 `Responses` 属性，来获取所有接收者的回复内容。

下面有一个简单的例子，`MainViewModel` 发送一个 `CollectionRequestMessage<int>` 消息，并等待一系列 `SubViewModel` 的回复，从而计算当前活跃的子视图模型的数量：

```csharp
// 主视图模型负责发送消息并统计活跃的子视图模型数量
class MainViewModel : ObservableObject
{
    public void CheckSubsStatus()
    {
        var response = WeakReferenceMessenger.Default.Send<CollectionRequestMessage<int>>(new());
        Console.WriteLine($"Alive count: {response.Count()}");
    }
}

// 子视图模型负责接收消息并回复自己的状态
class SubViewModel : ObservableRecipient, IRecipient<CollectionRequestMessage<int>>
{
    public bool IsAlive { get; set; }

    public void Receive(CollectionRequestMessage<int> message)
    {
        message.Reply(IsAlive ? 1 : 0);
    }
}
// 
var mainVM = new MainViewModel();
var sub1 = new SubViewModel { IsActive = true };
var sub2 = new SubViewModel { IsActive = true };
var sub3 = new SubViewModel { IsActive = false };
var sub4 = new SubViewModel { IsActive = true };
var sub5 = new SubViewModel { IsActive = false };
mainVM.CheckSubsStatus();
```

运行结果会得到当前活跃的子视图模型数量为 3。

此外，`CollectionRequestMessage` 还提供了一个异步版本 `AsyncCollectionRequestMessage<T>`，用于异步地接收多个回复，并且提供了 `CancellationToken`，用于取消等待回复的操作。

## PropertyChangedMessage

`PropertyChangedMessage` 是一个用于通知属性发生变化的消息类型，它包含一个 `PropertyName` 属性，用于存储属性的名称。如果希望发送一个用于通知某个属性发生变化的消息，可以使用 `PropertyChangedMessage`。

这个类通常配合 `ObservableRecipient` 的 `Broadcast` 方法使用，详见 [相关章节](../ComponentModel/ObservableRecipient.md)。
