---
comments: true
---

# `IRecipient<>` 接口

`IRecipient<>` 接口是一个泛型接口，它的泛型参数是一个 `TMessage` 类型的消息。这个接口定义了一个 `Receive` 方法，用于接收消息。我们可以通过实现这个接口来接收消息，形如：

```csharp
public class MyViewModel : ObservableObject, IRecipient<MyMessage>
{
    public void Receive(MyMessage message)
    {
        // 处理消息
    }
}
```

如果这个类需要接收多种 `TMessage` 类型的消息，那么可以实现多个 `IRecipient<>` 接口。

在实现了这样的接口后，我们不需要使用传统的方式去注册这些接收消息的方法，而是可以简单地调用 `Messenger.RegisterAll(this)` 方法，将当前类实现的所有 `IRecipient<>` 接口的方法注册到信使对象中。类似地，我们可以调用 `Messenger.UnregisterAll(this)` 方法来注销这些方法。

例如，这里我们使用 `WeakReferenceMessenger.Default` 作为信使对象。那么，我们的 `ViewModel` 类可以这样实现：

```csharp
public class ViewModel : ObservableRecipient, IRecipient<Message>
{
    public ViewModel()
    {
        Messenger.RegisterAll(this);
    }

    public void Receive(Message message)
    {
        // 处理消息
    }
}
```

!!! tip
    `RegisterAll` 方法还包含一个重载，允许传入一个 `Token` 对象，用于指定消息的频道。类似地，`UnregisterAll` 方法也包含一个重载，允许传入一个 `Token` 对象。但如果不传入 `Token` 对象，则默认为不考虑频道，注销当前对象注册的全部消息。

!!! note
    `RegisterAll` 方法写在 [`IMessengerExtensions.cs`](https://github.com/CommunityToolkit/dotnet/blob/main/src/CommunityToolkit.Mvvm/Messaging/IMessengerExtensions.cs) 中，是对 `IMessenger` 对象的扩展。它的大致逻辑是，通过反射，找到传入的对象中实现了 `IRecipient<>` 接口的所有方法，并将它们注册到信使对象中。

    `UnregisterAll` 方法的实现逻辑与 `RegisterAll` 类似，只是将注册改为注销。这个方法是写在具体的类的实现中的，比如 [`WeakReferenceMessenger` 类](https://github.com/CommunityToolkit/dotnet/blob/main/src/CommunityToolkit.Mvvm/Messaging/WeakReferenceMessenger.cs)。