# Messages

工具包中的 `IMessenger` 在 `Send` 或 `Register` 时并没有要求消息的种类满足任何条件（比如继承自某个基类），所以完全可以自由地实现任何消息类型。但是，工具包仍然为我们提供了几个常用的消息类型，包括：

- `ValueChangedMessage`
- `PropertyChangedMessage`
- `RequestMessage`

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

## PropertyChangedMessage

`PropertyChangedMessage` 是一个用于通知属性发生变化的消息类型，它包含一个 `PropertyName` 属性，用于存储属性的名称。如果希望发送一个用于通知某个属性发生变化的消息，可以使用 `PropertyChangedMessage`。

这个类通常作为 `INotifyPropertyChanged` 接口所实现的属性发生变化的通知的延申，用于额外通知其他的 ViewModel。

## RequestMessage

这是一个特殊的消息类型。如果 `Send` 方法发送的是这个类（或它的子类），那么 `Send` 方法将拥有一个返回值，这个返回值就是消息的接收者回复的消息。此时，消息的接收者也能够通过消息上的 `Reply` 方法回复消息的发送者。比如：

```csharp
public class MyRequestMessage : RequestMessage<string>
{
    public MyRequestMessage(string value) : base(value)
    {
    }
}

// 发送消息
var response = WeakReferenceMessenger.Default.Send(new MyRequestMessage("Hello World!"));

// 接收消息
WeakReferenceMessenger.Default.Register<MyRequestMessage>(this, message =>
{
    Console.WriteLine(message.Value);
    message.Reply("Hello World!");
});
```