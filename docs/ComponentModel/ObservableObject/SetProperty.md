# SetProperty

`SetProperty` 方法用于设置属性值，并在属性值发生变化时触发 `PropertyChanged` 事件。该方法的底层其实就是在调用 `OnPropertyChanged`。这里是一个最简单的 `SetProperty` 方法的实现：

```csharp
protected bool SetProperty<T>([NotNullIfNotNull(nameof(newValue))] ref T field, T newValue, [CallerMemberName] string? propertyName = null)
{
    // 采用默认的 Comparer 来比较新旧值是否相等
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

!!! note "`SetProperty` 方法的多种重载"
    `SetProperty` 方法有非常多的重载。仅在 `ObservableObject` 中就包含六种；而在它的两个子类 `ObservableRecipient` 和 `ObservableValidator` 中，又各自包含更多的重载，分别提供了各自的额外功能（`Recipient` 额外提供了通知功能，`ObservableValidator` 额外提供了验证功能）。这些重载会在相应的章节进行介绍。

## 函数传参

该方法有六个方法重载，且不同的重载对应的传参也不尽相同。这里简单介绍一下这些方法涉及到的参数的含义及作用：

1. `ref T field`：传入一个类中字段的引用，用于存储属性值。通常情况下，适用于一个完整属性（即包含背后的字段，以及属性的 `getter` 和 `setter`）。只需要传入这个字段的引用，就能够实现在 `SetProperty` 方法内部修改字段的值。
2. `T oldValue`：传入旧值（通常）用于判断值是否发生变化（如果没有，则通常认为不需要触发 `PropertyChanged` 事件）。
3. `T newValue`：传入新值（通常是 `setter` 方法中的 `value`）。如果新值和旧值不相等，则会触发 `PropertyChanged` 事件。
4. `IEqualityComparer<T> comparer`：传入一个比较器，用于比较上面提到的新旧值是否相等。如果不传入，则默认使用 `EqualityComparer<T>.Default` 来比较。
5. `Action<T> callback`：传入一个回调函数，用于在属性值发生变化时执行一些额外的操作。
6. `TModel model`：传入一个对象（通常是一个被当前的 ViewModel 所包装 Model）。这个用法会在下面详细介绍。

## 函数返回值

`SetProperty` 方法的返回值是一个 `bool` 类型的值，用于表示属性值是否发生变化。如果属性值发生变化，则返回 `true`，否则返回 `false`。所以，如果希望在属性值发生变化时执行一些额外的操作，可以通过判断 `SetProperty` 方法的返回值来实现。比如：

```csharp
private string _name;

public string Name
{
    get => _name;
    set
    {
        if (SetProperty(ref _name, value, nameof(Name)))
        {
            // 属性值发生变化时，执行一些额外的操作
        }
    }
}
```