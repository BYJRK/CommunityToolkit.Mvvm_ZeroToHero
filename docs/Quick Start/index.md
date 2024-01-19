# 快速开始

## 安装 CommunityToolkit.Mvvm

可以在程序包管理器中搜索 `CommunityToolkit.Mvvm` 并安装，或者使用以下命令安装

```bash
dotnet add package CommunityToolkit.Mvvm
```

即可开始使用。

## 使用传统方式创建 ViewModel

`ObservableObject` 是工具包提供的一个实现了 `INotifyPropertyChanged` 接口的基类。所以 ViewModel 直接继承这个类即可。

```c# title="MyViewModel.cs"
using CommunityToolkit.Mvvm.ComponentModel;

class MyViewModel : ObservableObject
{
    private string _firstName;
    public string FirstName
    {
        get => _firstName;
        set 
        {
            if (_firstName == value)
                return;
            
            _firstName = value;
            OnPropertyChanged(); // (1)!
        }
    }

    private string _lastName;
    public string LastName
    {
        get => _lastName;
        set
        {
            if (SetProperty(ref _lastName, value)) // (2)!
            {
                OnPropertyChanged(nameof(FullName));
            }
        } 
    }

    public string FullName => $"{FirstName} {LastName}";

    public RelayCommand SayHelloCommand { get; }

    public MyViewModel()
    {
        SayHelloCommand = new RelayCommand(SayHello);
    }

    private void SayHello()
    {
        // ...
    }
}
```

1. 这里因为 `OnPropertyChanged()` 方法中的参数使用了 `[CallerMemberName]` 特性，所以并不需要为方法传递属性名（通常写作 `nameof(Name)`）。
2. `SetProperty()` 方法基本上相当于 `Name` 属性的 `setter` 中的代码，是一个更简便的写法。它有一个 `bool` 类型的返回值，表示属性值是否发生了变化。

## （推荐👍）借助源生成器创建 ViewModel

工具包真正强大的地方在于源生成器。如果使用这一功能，那么上面的代码将可以简化为：

```c# title="MyViewModel.cs"
using CommunityToolkit.Mvvm.ComponentModel;

partial /* (1)! */ class MyViewModel : ObservableObject
{
    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(FullName))] // (2)!
    private string _firstName; // (3)!
    
    [ObservableProperty]
    [NotifyPropertyChangedFor(nameof(FullName))]
    private string _lastName;

    public string FullName => $"{FirstName} {LastName}";

    [RelayCommand]
    private void SayHello()
    {
        // ...
    }
}
```

1. 源生成器会在后台生成类中的其他代码，所以需要标记为 `partial`。
2. 这个特性能够在 `setter` 中额外添加对于 `FullName` 属性的通知。
3. 源生成器可以智能地识别各种常见的字段命名习惯，比如 `name`、`_name`、`m_name` 等等。

简单来说，标记了 `[ObservableProperty]` 的字段，在后台会自动生成一个首字母大写的属性，并实现相应的 `getter` 及 `setter` 方法；标记了 `[RelayCommand]` 的方法，在后台会自动生成一个名称为 `<MethodName>Command` 的 `RelayCommand` 类型的属性。

!!! tip "温馨提示"
    请尽管放心，源生成器远比你想象得要强大，且考虑十分周到，几乎可以满足所有你的特殊需求。

!!! warning "源生成器对 DOTNET SDK 版本的要求"
    源生成器最适合的开发环境是 .NET 6.0+。如果是较低版本，或者是 .NET Framework 项目，可能会出现问题。关于如何在 .NET Framework 项目中使用源生成器，可以参考[我的这期视频](https://www.bilibili.com/video/BV188411g7ox/)。

## 创建 View 并绑定

```xml title="MainWindow.xaml"
<Window x:Class="MyApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:viewmodels="clr-namespace:MyApp.ViewModels"
        Title="MyApp" Height="450" Width="800">
    <Window.DataContext>
        <viewmodels:MyViewModel/>
    </Window.DataContext>
    <StackPanel>
        <TextBox Text="{Binding FirstName}"/>
        <TextBox Text="{Binding LastName}"/>
        <TextBlock Text="{Binding FullName}"/>
        <Button Command="{Binding SayHelloCommand}"/>
    </StackPanel>
</Window>
```

然后就可以运行查看效果了。