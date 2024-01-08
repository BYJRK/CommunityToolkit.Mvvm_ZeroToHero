# ObservableObject

[`ObservableObject`](https://github.com/CommunityToolkit/dotnet/blob/main/src/CommunityToolkit.Mvvm/ComponentModel/ObservableObject.cs "ObservableObject 源代码") 是工具包为我们提供的一个实现了 [`INotifyPropertyChanged`](https://source.dot.net/#System.ObjectModel/System/ComponentModel/INotifyPropertyChanged.cs,fd4b42d7e29d53e5 "INotifyPropertyChanged 源代码") 接口的基类[^1]。

它拥有两个子类：

- `ObservableValidator`
- `ObservableRecipient`

分别在基类的基础上提供了一些额外的功能。这些内容将会在它们各自的文档中介绍。

!!! note "ViewModelBase"
    实际使用时，通常建议写一个 `ViewModelBase` 来继承这个基类，然后所有的 ViewModel 都继承这个 `ViewModelBase`。这样做的好处是，可以在 `ViewModelBase` 中实现一些通用的逻辑；此外，还有助于后续借助反射等来实现 `ViewModelLocator` 等功能（因为并不是只有严格意义上的 ViewModel 才能继承这个基类；任何想要具有通知功能的类都可以继承）。

[^1]: 实际上它实现了不仅仅这个接口，还有 `INotifyPropertyChanging`。这个接口的作用是在属性值发生变化之前通知。但是在实际使用中，我们很少会用到这个接口，因此这里不做介绍。

## ObservableObject 类提供的方法

`ObservableObject` 除了实现了 `INotifyPropertyChanged` 接口之外，还提供了一些方法来帮助我们实现通知功能。比如：

- `OnPropertyChanged`（拥有两个方法重载）
- `SetProperty`（包含六个方法重载）
- `SetPropertyAndNotifyOnCompletion`（包含五个重载，用于 `TaskNotifier`）

!!! tip "实际使用"
    实际上，因为有源生成器（Source Generator），大多数情况下，我们并不需要手动去调用这些方法，甚至不需要书写属性。关于这部分内容，请参见 [源生成器](../../Source%20Generator/index.md)