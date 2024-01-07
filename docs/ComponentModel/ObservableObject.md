# ObservableObject

[`ObservableObject`](https://github.com/CommunityToolkit/dotnet/blob/main/src/CommunityToolkit.Mvvm/ComponentModel/ObservableObject.cs "ObservableObject 源代码") 是工具包为我们提供的一个实现了 [`INotifyPropertyChanged`](https://source.dot.net/#System.ObjectModel/System/ComponentModel/INotifyPropertyChanged.cs,fd4b42d7e29d53e5 "INotifyPropertyChanged 源代码") 接口的基类[^1]。

它拥有两个子类：

- `ObservableValidator`
- `ObservableRecipient`

分别在基类的基础上提供了一些额外的功能。这些内容将会在它们各自的文档中介绍。

!!! NOTE "提示"
    实际使用时，通常建议写一个 `ViewModelBase` 来继承这个基类，然后所有的 ViewModel 都继承这个 `ViewModelBase`。这样做的好处是，可以在 `ViewModelBase` 中实现一些通用的逻辑；此外，还有助于后续借助反射等来实现 `ViewModelLocator` 等功能（因为并不是只有严格意义上的 ViewModel 才能继承这个基类；任何想要具有通知功能的类都可以继承）。

[^1]: 实际上它实现了不仅仅这个接口，还有其它一些不太常用的，比如 `INotifyPropertyChanging` 等。

## 类提供的方法

`ObservableObject` 除了实现了 `INotifyPropertyChanged` 接口之外，还提供了一些方法来帮助我们实现通知功能。比如：

- OnPropertyChanged（拥有两个方法重载）
- SetProperty（包含六个方法重载）

### OnPropertyChanged

`OnPropertyChanged` 方法用于触发 `PropertyChanged` 事件。它有两个方法重载：

- `OnPropertyChanged([CallerMemberName] string? propertyName = null)`
- `OnPropertyChanged(PropertyChangedEventArgs args)`

!!! NOTE "CallerMemberName 特性"
    `[CallerMemberName]` 特性是 C# 5.0 中引入的。它可以用于方法的参数中，表示这个参数的值会在编译时自动传入方法调用处的方法名。如果在一个属性的 `setter` 方法中使用，那么将等价于传入了属性的名称。

这两个方法的区别在于，前者只需要传入属性名，而后者需要传入一个 `PropertyChangedEventArgs` 对象。这个对象包含了属性名和属性值，以及一些其它信息。这两个方法的实现都是一样的，都是调用 `RaisePropertyChanged` 方法来触发事件。

如果只依靠这个方法，那么可以这样来实现一个简单的 ViewModel：

=== "简易版"

    ```csharp
    class MyViewModel : ObservableObject
    {
        private string _name;
        public string Name
        {
            get => _name;
            set
            {
                _name = value;
                OnPropertyChanged(); // (1)!
            }
        }
    }
    ```

    1.  这里因为 `[CallerMemberName]` 特性，所以并不需要为方法传递属性名（通常写作 `nameof(Name)`）。

=== "完整版"

    ```csharp
    class MyViewModel : ObservableObject
    {
        private string _name;
        public string Name
        {
            get => _name;
            set
            {
                // 检查值是否发生变化
                if (_name == value /* (1)! */) return;

                // 触发属性值即将发生变化的事件
                OnPropertyChanging();

                // 设置属性值
                _name = value;

                // 触发属性值已经发生变化的事件
                OnPropertyChanged();
            }
        }
    }
    ```

    1. 这里也可以使用这样的方式：`EqualityComparer<T>.Default.Equals(field, newValue)`

### SetProperty

`SetProperty` 方法用于设置属性值，并在属性值发生变化时触发 `PropertyChanged` 事件。它有六个方法重载，这里主要介绍一下这些方法涉及到的一些参数：

1. `ref T field`：传入一个类中字段的引用，用于存储属性值。通常情况下，适用于一个完整属性（即包含背后的字段，以及属性的 `getter` 和 `setter`）。只需要传入这个字段的引用，就能够实现在 `SetProperty` 方法内部修改字段的值。
2. `T oldValue`：个别方法重载使用到了。传入旧值通常用于判断值是否发生变化（如果没有，则通常认为不需要触发 `PropertyChanged` 事件）。
3. `T newValue`：传入新值（通常是 `setter` 方法中的 `value`）。如果新值和旧值不相等，则会触发 `PropertyChanged` 事件。
4. `IEqualityComparer<T> comparer`：传入一个比较器，用于比较上面提到的新旧值是否相等。如果不传入，则默认使用 `EqualityComparer<T>.Default` 来比较。
5. `Action<T> callback`：传入一个回调函数，用于在属性值发生变化时执行一些额外的操作。
6. `TModel model`：传入一个对象（通常是一个被当前的 ViewModel 所包装 Model）。这个用法会在下面详细介绍。