# Model 与 ViewModel 之间的关系

Model 与 ViewModel 之间的关系一直是 MVVM 模式中最容易让人混淆的地方。从最基本的概念来说，Model 是数据，ViewModel 是数据的展示方式。

但是这个概念并不是很容易理解，因为在实际的开发中，我们经常会有意无意地把对数据的获取、处理、展示等功能都放在 ViewModel 中，这样的话，ViewModel 就不仅仅是数据的展示方式了。

> 关于这部分内容，可以看看我的这期视频：[从 ASP.NET 及 EFCore 出发探讨 MVVM 中的 Model 究竟是什么](https://www.bilibili.com/video/BV1Vz421f7Do/)。

我们依旧以前面提到的学生信息管理系统为例。

## Model

首先，为了和数据库交互（这里我们使用关系型数据库，并借助 EFCore 作为 ORM 框架）。因此，首先我们有了如下的数据相关的类：

- Student.cs
- DbContext.cs
- Repository.cs

其中，`Student.cs` 是一个非常纯粹的 POCO 类，表示学生的信息。它并不包含任何额外的功能，比如加载、保存、验证、甚至通知等功能。它唯一的作用是在 `DbContext` 中作为数据表的映射类。它的代码大致如下：

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    // 其他属性
}
```

然后我们来看 EFCore 的 `DbContext` 类，它的作用是连接数据库，并提供对数据库的操作。它的代码大致如下：

```csharp
public class StudentDbContext : DbContext
{
    public DbSet<Student> Students { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("your_connection_string");
    }
}
```

现在有了 `DbContext` 后，虽然我们可以直接借助它来操作数据库，但释放太多细节给 ViewModel 可能并不是一个好主意。所以通常我们的做法是使用仓储模式，即创建一个 `Repository` 类来封装对数据库的操作。通常我们还会提供接口，从而封装不需要暴露的功能，以及支持数据模拟（Mock）等，但这里我们不探讨。它的代码大致如下：

```csharp
public class StudentRepository : IStudentRepository
{
    private readonly StudentDbContext _context;

    public StudentRepository(StudentDbContext context)
    {
        _context = context;
    }

    public IEnumerable<Student> GetAllStudents()
    {
        return _context.Students.ToList();
    }

    public void AddStudent(Student student)
    {
        _context.Students.Add(student);
        _context.SaveChanges();
    }

    // 其他操作
}
```

## ViewModel

现在，为了能够让用户在界面上操作数据，我们就需要提供视图模型了。比如想要展示全部学生信息时，我们可以创建一个 `MainViewModel` 类，它的代码大致如下：

```csharp
public partial class MainViewModel : ViewModelBase
{
    // 依赖注入
    private readonly IStudentRepository _studentRepository;

    [ObservableProperty]
    private ObservableCollection<Student> _students;

    public MainViewModel(IStudentRepository studentRepository)
    {
        _studentRepository = studentRepository;
        LoadStudents();
    }

    private void LoadStudents()
    {
        var students = _studentRepository.GetAllStudents();
        Students = new ObservableCollection<Student>(students);
    }
    
}
```

!!! warning "注意"
    上面的代码已经使用了社区工具包的相关功能。这里假定你已经了解了它的大致用法。否则建议先阅读 [快速开始](../Quick%20Start/index.md) 中的相关内容。

然后，我们就可以将这里的 `Students` 属性绑定到视图中的 `DataGrid` 上了：

```xml
<DataGrid ItemsSource="{Binding Students}">
    <DataGrid.Columns>
        <DataGridTextColumn Header="Id" Binding="{Binding Id}" />
        <DataGridTextColumn Header="Name" Binding="{Binding Name}" />
        <DataGridTextColumn Header="Age" Binding="{Binding Age}" />
    </DataGrid.Columns>
</DataGrid>
```

其实到目前为止，相信绝大多数读者都可以明确地分清上面的类分别属于 MVVM 的哪个部分。接下来我们将引入一些不太好区分的部分。

## ViewModel 还是 Model？

现在我们不想局限于只展示学生的信息，而是想要提供一个可以添加学生信息的功能。于是我们引入了相应的 View 及 VM，即：

- AddStudentForm.xaml（及 AddStudentForm.xaml.cs）
- AddStudentFormViewModel.cs

然后我们可能会发现，在 `AddStudentFormViewModel` 中，我们需要提供一些属性来绑定到视图上，并且这些属性和前面属于 Model 的 `Student` 类高度相似。那么此时我们是否可以将 `Student` 类直接作为 `AddStudentFormViewModel` 的属性呢？比如：

```csharp
public partial class AddStudentFormViewModel : ViewModelBase
{
    [ObservableProperty]
    private Student _student;

    public AddStudentFormViewModel()
    {
        Student = new Student();
    }

    [RelayCommand]
    private void AddStudent()
    {
        // 通常这里会有数据校验相关的逻辑
        // 这里我们可以直接使用 Student 属性来添加学生
        _studentRepository.AddStudent(Student);
    }
}
```

在上面的代码中，我们直接将 `Student` 类作为了 `AddStudentFormViewModel` 的属性。并且这样相当省事。但这就会产生一个疑问：一般情况下，我们认为 Model 是和数据库交互的，而 VM 是和视图交互的，但这里它的属性却直接被绑定到了视图上。这样的话，`Student` 类似乎就变成了 VM 的一部分了。

如果现在我们对 `Student` 类的要求还并不高，那么它继续被认为是 Model 似乎并没有什么问题。但如果我们想引入更多功能呢？比如通知、校验等：

```csharp
public partial class Student : ObservableValidator
{
    [ObservableProperty]
    [Required]
    private string _name;

    [ObservableProperty]
    [Range(1, 100)]
    private int _age;

    // 其他属性
}
```

那么它还算是 Model 吗？

相信很多读者会认为，在一个 VM 中，如果有一个集合，那么通常情况下，集合的元素就是 Model。比如我们在展示学生列表时，集合的元素就是 `Student`。在这个例子中确实是成立的。但如果我们希望这些原本纯粹的 Model 具有额外的功能，那就不再纯粹了。事实上，此时它们已经变成了 VM。

可问题是，虽然它们此时被定义为 VM，它们却看起来很像 Model，至少和它们背后的那个 Model 类几乎看起来具有相当明显的映射关系。这也通常是新手感到迷惑的地方。

所以这里正确的实现方式是，为原本的 `Student` 类添加一个 `StudentViewModel` 类，或者通常我们还会叫它 Wrapper 类。它的代码大致如下：

```csharp
public partial class StudentViewModel : ObservableValidator
{
    [ObservableProperty]
    private string _name;

    [ObservableProperty]
    private int _age;

    // ...

    public StudentViewModel(Student student)
    {
        Name = student.Name;
        Age = student.Age;
        // 其他属性
    }
}
```

<!-- 或者我们还可以借助工具包的功能来直接将它和一个 Model 实例建立关系。这一部分可以参考 [](../ComponentModel/ObservableObject/index.md#) -->

这样，我们的代码逻辑就变成了：

读取：从仓储获取 `Student` 实例 -> 将其转换为 `StudentViewModel` 实例 -> 绑定到视图上

保存：从视图获取 `StudentViewModel` 实例 -> 将其转换为 `Student` 实例 -> 交给仓储进行保存

虽然着看起来可能有点多此一举，尤其是我们的需求比较简单，几乎可以直接将 Model 用作表单的 VM 时。但这样做却是推荐的，因为它可以使我们的项目结构更加清晰，并且在后期扩展时也会更加方便。否则，只要你有超出 Model 本身的任何需求，你都几乎不可避免地需要对 Model 进行修改，进而污染了 Model 的纯粹性。
