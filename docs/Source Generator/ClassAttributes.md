---
comments: true
---

# 适用于类的特性

虽然并不多，但是工具包依然为我们提供了一些可以用于（分部）类的特性，包括：

- `ObservableObject`
- `ObservableRecipient`
- `INotifyPropertyChanged`

它们看起来与工具包的几个基类，以及 INPC 接口完全同名（除了它们实际的名称是以 `Attribute` 结尾的）。它们的作用也非常直白，就是让当前类实现 INPC 接口，并为当前类添加这些类的功能，比如：

- `ObservableObject` 会使当前类拥有 `OnPropertyChanged`、`SetProperty` 等方法
- `ObservableRecipient` 会使当前类拥有 `Broadcast` 等方法
- `INotifyPropertyChanged` 会使当前类添加最基本的 `OnPropertyChanged` 方法

（目前工具包并未提供 `ObservableValidator` 特性）

这些特性的作用是为一个不便于改写的类（如数据模型类）添加功能，而不需要改写这个类。

!!! note
    实际上，这些特性并不非常实用，因为：

    1. 它们要求类必须是 `partial` 的，而有的时候我们并没有机会让这些类成为分部类
    2. 你必须有机会为这个类添加特性，而有时候这个类可能来自于第三方库
    3. 如果你需要用这种方式使一个类拥有通知功能，**那么这个类本身的设计以及功能就可能已经存在问题了**。这种情况下，通常应该考虑的做法是添加 DTO，或借助一些方法包装这个类。可以参考我的 [这期视频](https://www.bilibili.com/video/BV1Vz421f7Do/)。