---
comments: true
---

# ObservableRecipient

!!! tip 提示
    `ObservableRecipient`（以下简称 `OR`）是 `ObservableObject` 的子类，它提供了一些额外的功能，用于实现一个可接收消息的对象。在学习这个类的用法之前，建议先学习并掌握 [`Messenger`](../Messengers/index.md) 的用法。

## 概述

`OR` 因为是 `ObservableObject` 的子类，所以拥有基类的全部功能。在此基础上，它又增加了一些新的属性及方法，包括但不限于：

- `SetProperty`：`ObservableObject` 本身包含多种 `SetProperty` 方法的重载，详见 [SetProperty](ObservableObject/SetProperty.md)。在此基础上，`OR` 为其中的一些重载额外添加了一个参数：`bool broadcast`，表示是否广播属性值的变化。
- `Messenger`：该类所使用的信使对象。它是一个 `IMessenger` 类型的属性，用于发送和接收消息。
- `Broadcast`：用于广播属性值的变化。具体来说，它会新建一个 `PropertyChangedMessage<>` 类型的消息，并通过类的 `Messenger` 发送出去。
- `IsActive`：表示当前对象是否处于激活状态的属性。当它被设为 `true` 时，会调用 `OnActivated` 方法；当它被设为 `false` 时，会调用 `OnDeactivated` 方法。
- `OnActivated`：在对象被激活时调用。它的作用是调用类的 `Messenger.RegisterAll(this)` 方法，注册当前类实现的所有 `IRecipient<>` 接口的对应方法。
- `OnDeactivated`：在对象被停用时调用。它的作用是调用类的 `Messenger.UnregisterAll(this)` 方法，注销当前类注册的全部消息。

## 信使（Messenger）

从上面的一些方法可以看出，`OR` 拥有一个名为 `Messenger` 的信使对象。那么这个对象是怎么来的呢？

通过观察 `OR` 的构造函数可以发现，它拥有两个构造函数，分别是：

- 无参构造函数：会将 `WeakReferenceMessenger.Default` 作为默认信使对象赋值给 `Messenger` 属性
- 有参构造函数：需要传入一个 `IMessenger` 对象，并将其赋值给 `Messenger` 属性

所以不难看出，除非显式声明，否则 `OR` 的 `Messenger` 属性默认是 `WeakReferenceMessenger.Default`。这一设计非常巧妙：

1. 当我们不需要关注整个项目中的所有 `OR` 类型对应的信使对象的实现时，它全部默认采用了 `WeakReferenceMessenger.Default`；
2. 当我们需要进行定制，尤其是使用一些 `IoC` 容器进行这一实现时，我们可以为 `IMessenger` 接口类型指定一个具体的实现（如 `StrongReferenceMessenger.Default`），从而实现对于所有 `OR` 类型的信使对象的定制。

## `IRecipient` 接口的使用

在 [IRecipient](../Messengers/IRecipient.md) 章节中，我们介绍了 `IRecipient<>` 接口的用法。这里，`OR` 当然也可以（并且推荐）使用这种方法来管理当前类需要注册哪些消息类型。为此，`OR` 特意提供了一些额外的功能，方便我们使用这一机制。

具体来说，当调用 `OnActivated` 方法时，后台会调用 `Messenger.RegisterAll(this)` 方法，注册当前类实现的所有 `IRecipient<>` 接口的对应方法。而当调用 `OnDeactivated` 方法时，会调用 `Messenger.UnregisterAll(this)` 方法，注销当前类注册的全部消息。

更重要的是，我们并不需要显式调用这些方法，而是可以非常简单地通过设置 `IsActive` 属性地值来实现。当 `IsActive` 属性被设为 `true` 时，会调用 `OnActivated` 方法；当 `IsActive` 属性被设为 `false` 时，会调用 `OnDeactivated` 方法。

所以，一个典型的 `OR` 对象的声明和使用方式如下：

```csharp
public class MyViewModel : ObservableRecipient, IRecipient<MyMessage>
{
    public MyViewModel()
    {
        IsActive = true;
    }

    public void Receive(MyMessage message)
    {
        // 处理消息
    }
}
```

## `IsActive` 属性

虽然 `IsActive` 属性的作用已经足够强大，但很快我们还是会发现一些小问题，比如我们希望定制当前 `OR` 类所使用的频道（Token）。这时候该怎么做呢？

观察发现，`OR` 的 `OnActivated` 方法和 `OnDeactivated` 方法都是 `protected virtual` 的，这意味着我们可以在子类中重写这两个方法，从而实现我们的需求。比如，我们希望当前类关注的频道为 `"MyToken"`，那么我们可以这样实现：

```csharp
public class MyViewModel : ObservableRecipient, IRecipient<MyMessage>
{
    public MyViewModel()
    {
        IsActive = true;
    }

    public void Receive(MyMessage message)
    {
        // 处理消息
    }

    protected override void OnActivated()
    {
        // 这里不应当调用 base.OnActivated() 方法，因为它会不考虑频道，并注册所有消息
        Messenger.RegisterAll(this, "MyToken");
    }

    protected override void OnDeactivated()
    {
        Messenger.UnregisterAll(this, "MyToken");
    }
}
```

!!! tip
    实际使用中，通常我们不需要覆写 `OnDeactivated` 方法，因为如果不提供频道，那么 `UnregisterAll` 方法会默认注销当前对象注册的全部消息（不考虑频道），而这通常正是我们所期待的效果。

## 广播功能（Broadcast）

在 [Messages](../Messengers/Messages.md) 章节中，我们介绍了一些工具包自带的消息类型，其中就包括 `PropertyChangedMessage<>` 这一类型。这个消息类型的作用是用于通知属性值的变化。比如现在有另外一个类，它会关注当前 `OR` 对象中某个或某些属性的变化，那么我们可以通过传递 `PropertyChangedMessage<>` 类型的消息来实现这一功能。不仅如此，我们还可以考虑定制频道，从而实现更加精细的控制。

为此，`OR` 提供了一个名为 `Broadcast` 的方法，用于广播属性值的变化。具体来说，它会新建一个 `PropertyChangedMessage<>` 类型的消息，并通过类的 `Messenger` 发送出去。它的源代码如下：

```csharp
protected virtual void Broadcast<T>(T oldValue, T newValue, string? propertyName)
{
    PropertyChangedMessage<T> message = new(this, propertyName, oldValue, newValue);

    _ = Messenger.Send(message);
}
```

很快我们又会发现，如果我们希望定制当前 `OR` 对象所使用的频道，那么我们还需要重写 `Broadcast` 方法，从而实现我们的需求。如下：

```csharp
protected override void Broadcast<T>(T oldValue, T newValue, string? propertyName)
{
    PropertyChangedMessage<T> message = new(this, propertyName, oldValue, newValue);

    _ = Messenger.Send(message, "MyToken");
}
```

如果想要在属性值发生变化时广播，除了可以调用特殊的 `SetProperty` 方法外，还可以使用源生成器，为字段额外添加一个 `NotifyPropertyChangedRecipients` 特性。

```csharp
private string _name;

public string Name
{
    get => _name;
    set => SetProperty(ref _name, value, true);
}

[ObservableProperty]
[NotifyPropertyChangedRecipients]
private int _age;
```

这一部分详见 [与字段相关的源生成器特性](../Source%20Generator/FieldAttributes.md#_5)。