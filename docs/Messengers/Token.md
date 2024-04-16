---
comments: true
---

# 消息频道 Token

通过前面的例子会发现，消息的订阅与发布是基于消息的类型的。当发送了一个指定类型的消息，那么任何订阅了这一类型的接收者都会收到消息。

但这样可能并不是我们想要的效果，比如我们希望消息只发送给指定的一部分接收者，而不是所有订阅了这一类型的接收者。在这种情况下，我们当然不希望为此而声明大量不同的消息类型。这时，我们可以使用频道来解决这个问题。

!!! info "关于“频道”这一概念"
    “Token”这个单词通常翻译为“令牌”，但在这里，我将其翻译为“频道”。这是因为“令牌”这个词在计算机领域有着多种含义，而“频道”这个词则更加贴近我们的使用场景。

## 使用方法

频道的使用方法非常简单，我们只需要给 `Send` 与 `Register` 两个方法分别添加一个参数即可。这里，我们选择了 `string` 类型的参数，也就是下面代码中出现的 `"token"` 参数。

!!! tip "频道参数的类型"
    用于标识频道的参数并不局限于字符串，**你可以使用任何类型的对象作为频道**。工具包对于 `TToken` 泛型类型的唯一要求是它实现了 `IEquatable` 接口，也就是说可以比较值是否相等。

=== "Program.cs"
    
    ```csharp
    var vm = new ViewModel();
    WeakReferenceMessenger.Default.Send(new Message("Hello"), "token");
    ```

=== "ViewModel.cs"
    
    ```csharp
    class ViewModel
    {
        public ViewModel()
        {
            WeakReferenceMessenger.Default.Register<Message, string>(this, "token",
                (_, m) => { Console.WriteLine(m.Content); });
        }
    }
    ```

=== "Message.cs"
    
    ```csharp
    class Message
    {
        public Message(string content)
        {
            Content = content;
        }

        public string Content { get; }
    }
    ```

## 不提供 token

如果我们在调用 `Send` 等方法时并没有指定频道，那么后台其实会最终调用一个包含 token 参数的 `Send` 方法，并传入一个“空的令牌”。准确地说，是 `default(Unit)`，其中 `Unit` 是工具包中提供的一个空的结构体。

=== "IMessengerExtensions.cs"

    ```csharp
    public partial class IMessengerExtensions
    {
        public static TMessage Send<TMessage>(this IMessenger messenger, TMessage message)
            where TMessage : class
        {
            ArgumentNullException.ThrowIfNull(messenger);
            ArgumentNullException.ThrowIfNull(message);

            return messenger.Send(message, default(Unit));
        }
    }
    ```

=== "Unit.cs"

    ```csharp
    internal readonly struct Unit : IEquatable<Unit>
    {
        public bool Equals(Unit other)
        {
            return true;
        }

        public override bool Equals(object? obj)
        {
            return obj is Unit;
        }

        public override int GetHashCode()
        {
            return 0;
        }
    }
    ```