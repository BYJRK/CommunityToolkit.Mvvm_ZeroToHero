# WeakReferenceMessenger & StrongReferenceMessenger

## 什么是 WeakReference？

在说 `WeakReferenceMessenger` 之前，我们先要搞清楚什么是弱引用（`WeakReference`）。

在 C# 中，我们可以通过 `new WeakReference(obj)` 来创建一个弱引用，这个弱引用会指向 `obj`，但是不会阻止 `obj` 被 GC 回收。

```csharp
var obj = new object();
var weakReference = new WeakReference(obj);
```

这一特性在 MVVM 中显得尤为重要。因为我们如果想要借助 `Messenger` 来实现 MVVM 中的消息传递，那么我们就需要在 `ViewModel` 中注册事件。一旦我们没有及时取消注册，那么这个 `ViewModel` 就无法被 GC 回收，这就会导致内存泄漏。而使用 `WeakReferenceMessenger` 就可以避免这个问题。

## WeakReferenceMessenger

在[前面的例子](./IMessenger.md#基本用法 "IMessenger 中的代码例子")中，我们使用 `WeakReferenceMessenger.Default` 来获取一个默认的 `WeakReferenceMessenger` 实例。这个实例是一个单例，我们可以在任何地方使用它。如果我们的项目比较简单，比如不需要考虑其他 Messenger 类型，且不使用 IoC 容器，那么我们可以直接使用这个单例。

## StrongReferenceMessenger

它与 `WeakReferenceMessenger` 的唯一区别在于，它使用强引用，因此消息接收者不会被 GC 回收。在用法上，它与 `WeakReferenceMessenger` 没有区别，因此不再赘述。

但是，在使用传统的强引用时，我们需要注意内存泄漏的问题。我们需要在代码中合适的位置调用 `Unregister` 方法，以取消注册。