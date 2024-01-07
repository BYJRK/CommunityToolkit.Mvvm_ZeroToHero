# OnPropertyChanged

`OnPropertyChanged` 方法用于触发 `PropertyChanged` 事件。它有两个方法重载：

- `OnPropertyChanged([CallerMemberName] string? propertyName = null)`
- `OnPropertyChanged(PropertyChangedEventArgs args)`

!!! NOTE "CallerMemberName 特性"
    `[CallerMemberName]` 特性是 C# 5.0 中引入的。它可以用于方法的参数中，表示这个参数的值会在编译时自动传入方法调用处的方法名。
    
    如果在一个属性的 `setter` 方法中使用，那么将等价于传入了属性的名称；如果在一个方法中使用，则等价于传入了这个方法的名称。

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