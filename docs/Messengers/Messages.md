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

## PropertyChangedMessage

`PropertyChangedMessage` 是一个用于通知属性发生变化的消息类型，它包含一个 `PropertyName` 属性，用于存储属性的名称。如果希望发送一个用于通知某个属性发生变化的消息，可以使用 `PropertyChangedMessage`。

这个类通常配合 `ObservableRecipient` 的 `Broadcast` 方法使用，详见 [相关章节](../ComponentModel/ObservableRecipient.md)。
