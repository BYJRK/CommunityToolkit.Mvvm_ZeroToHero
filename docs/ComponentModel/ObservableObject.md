# ObservableObject

`ObservableObject` 是工具包为我们提供的一个实现了 `INotifyPropertyChanged` 接口的基类[^1]。

它拥有两个子类：

- `ObservableValidator`
- `ObservableRecipient`

分别在基类的基础上提供了一些额外的功能。这些内容将会在它们各自的文档中介绍。

!!! NOTE "提示"
    实际使用时，通常建议写一个 `ViewModelBase` 来继承这个基类，然后所有的 ViewModel 都继承这个 `ViewModelBase`。这样做的好处是，可以在 `ViewModelBase` 中实现一些通用的逻辑；此外，还有助于后续借助反射等来实现 `ViewModelLocator` 等功能。

[^1]: 实际上它实现了不仅仅这个接口，还有其它一些不太常用的，比如 `INotifyPropertyChanging` 等。