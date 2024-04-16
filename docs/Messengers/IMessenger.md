---
comments: true
---

# `IMessenger` 接口

`IMessenger` 是工具包中的一个接口，规定了 Messenger 需要实现的一些方法，比如：

- `Register`：注册消息
- `Unregister`：取消注册
- `UnregisterAll`：取消所有注册
- `Send`：发送消息

## 实现了这一接口的类

工具包中提供了两个实现了 `IMessenger` 接口的类：

- `WeakReferenceMessenger`：使用弱引用，避免内存泄漏
- `StrongReferenceMessenger`：使用强引用，通常用于希望消息接收者不被 GC 回收的场景

一般来说，我们使用 `WeakReferenceMessenger` 即可。

## 基本用法

Messenger 的使用方法基本上分三步走：

1. 声明一个消息类型，或使用工具包内置的几种类型
2. 注册消息（`IMessneger.Register`）
3. 发送消息（`IMessenger.Send`）

下面是一个简单的例子：

=== "MessageSender"

    ```c#
    class MessageSender
    {
        private readonly IMessenger _messenger;

        public MessageSender(IMessenger messenger)
        {
            _messenger = messenger;
        }

        public void SendMessage()
        {
            // 发送消息
            _messenger.Send(new MyMessage { Content = "Hello, world!" });
        }
    }
    ```

=== "MessageReceiver"

    ```c#
    class MessageReceiver
    {
        private readonly IMessenger _messenger;

        public MessageReceiver(IMessenger messenger)
        {
            _messenger = messenger;

            // 注册指定类型的消息
            _messenger.Register<MyMessage>(this, OnMessageReceived);
        }

        void OnMessageReceived(object recipient, MyMessage message)
        {
            Console.WriteLine($"Message received from {recipient.GetType().Name}: {message.Content}");
        }
    }
    ```

=== "MyMessage"

    ```c#
    // 一个自定义消息类型
    class MyMessage
    {
        public string Content { get; init; } = "";
    }
    ```

然后我们实例化这些类：

```c# title="Program.cs"
IMessenger messenger = WeakReferenceMessenger.Default;
var receiver = new MessageReceiver(messenger);
var sender = new MessageSender(messenger);

sender.SendMessage();
```

!!! note
    这里我们使用了依赖注入的方式，将 `IMessenger` 注入到了 `MessageSender` 和 `MessageReceiver` 中。这样做的好处是，我们可以很方便地将 `IMessenger` 替换为其他实现了 `IMessenger` 接口的类，比如 `StrongReferenceMessenger`；同时，在类中，我们也不需要关心 `IMessenger` 的具体实现，只需要知道它是一个 `IMessenger`，以及它提供的方法即可。

运行即可看到效果：

```
Message received from MessageReceiver: Hello, world!
```

一些更复杂及灵活的用法详见后面的章节。