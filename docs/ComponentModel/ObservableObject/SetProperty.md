# SetProperty

`SetProperty` 方法用于设置属性值，并在属性值发生变化时触发 `PropertyChanged` 事件。它有六个方法重载，这里主要介绍一下这些方法涉及到的一些参数：

1. `ref T field`：传入一个类中字段的引用，用于存储属性值。通常情况下，适用于一个完整属性（即包含背后的字段，以及属性的 `getter` 和 `setter`）。只需要传入这个字段的引用，就能够实现在 `SetProperty` 方法内部修改字段的值。
2. `T oldValue`：个别方法重载使用到了。传入旧值通常用于判断值是否发生变化（如果没有，则通常认为不需要触发 `PropertyChanged` 事件）。
3. `T newValue`：传入新值（通常是 `setter` 方法中的 `value`）。如果新值和旧值不相等，则会触发 `PropertyChanged` 事件。
4. `IEqualityComparer<T> comparer`：传入一个比较器，用于比较上面提到的新旧值是否相等。如果不传入，则默认使用 `EqualityComparer<T>.Default` 来比较。
5. `Action<T> callback`：传入一个回调函数，用于在属性值发生变化时执行一些额外的操作。
6. `TModel model`：传入一个对象（通常是一个被当前的 ViewModel 所包装 Model）。这个用法会在下面详细介绍。

??? NOTE "SetProperty 的底层原理"
    `SetProperty` 方法的底层其实就是在调用 `OnPropertyChanged` 方法。这里是一个最简单的 `SetProperty` 方法的实现：

    ```csharp
    protected bool SetProperty<T>([NotNullIfNotNull(nameof(newValue))] ref T field, T newValue, [CallerMemberName] string? propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, newValue))
        {
            return false;
        }

        OnPropertyChanging(propertyName);

        field = newValue;

        OnPropertyChanged(propertyName);

        return true;
    }
    ```