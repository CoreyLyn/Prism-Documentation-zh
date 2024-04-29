# Commanding

除了提供对要在视图中显示或编辑的数据的访问之外，ViewModel 还可能定义一个或多个可由用户执行的操作或操作。用户可以通过 UI 执行的操作或操作通常定义为命令。命令提供了一种方便的方式来表示可以轻松绑定到 UI 中的控件的操作或操作。它们封装了实现操作或操作的实际代码，并有助于将其与其在视图中的实际视觉表示形式分离。

当命令与视图交互时，用户可以通过许多不同的方式直观地表示和调用命令。在大多数情况下，它们会因单击鼠标而调用，但也可以通过快捷键按下、触摸手势或任何其他输入事件来调用。视图中的控件是绑定到 ViewModel 命令的数据，以便用户可以使用控件定义的任何输入事件或手势来调用它们。视图中的 UI 控件与命令之间的交互可以是双向的。在这种情况下，可以在用户与 UI 交互时调用该命令，并且可以在启用或禁用基础命令时自动启用或禁用 UI。

ViewModel 可以将命令实现为命令对象（实现接口的 ICommand 对象）。视图与命令的交互可以通过声明方式定义，而无需在视图的代码隐藏文件中编写复杂的事件处理代码。例如，某些控件本身支持命令，并提供一个 Command 属性，该属性可以绑定到 ViewModel 提供的 ICommand 对象的数据。在其他情况下，命令行为可用于将控件与 ViewModel 提供的命令方法或命令对象相关联。

实现 ICommand 接口很简单。Prism 提供了此接口的 DelegateCommand 实现，您可以在应用程序中轻松使用。

?> DelegateCommand 可以在 Prism.Core NuGet 包中的 Prism.Commands 命名空间中找到。

Prism DelegateCommand 类封装了两个委托，每个委托都引用在 ViewModel 类中实现的方法。它通过调用这些委托来实现 ICommand 接口 Execute 的和 CanExecute 方法。在 DelegateCommand 类构造函数中指定 ViewModel 方法的委托。例如，下面的代码示例演示如何通过指定 OnSubmit 和 CanSubmit ViewModel 方法的委托来构造表示 Submit 命令的 DelegateCommand 实例。然后，该命令通过只读属性向视图公开，该属性返回对 DelegateCommand。

```cs
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand<object>(Submit, CanSubmit);
    }

    void Submit(object parameter)
    {
        //implement logic
    }

    bool CanSubmit(object parameter)
    {
        return true;
    }
}
```

在 DelegateCommand 对象上调用 Execute 方法时，它只是通过在构造函数中指定的委托将调用转发给 ViewModel 类中的方法。同样，调用该 CanExecute 方法时，将调用 ViewModel 类中的相应方法。构造函数中方法的 CanExecute 委托是可选的。如果未指定委托， DelegateCommand 则始终返回 true CanExecute 。

该 DelegateCommand 类是泛型类型。type 参数指定传递给 Execute and CanExecute 方法的命令参数的类型。在前面的示例中，命令参数的类型 object 为 。Prism 还提供了该 DelegateCommand 类的非泛型版本，供在不需要命令参数时使用，其定义如下：

```
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit, CanSubmit);
    }

    void Submit()
    {
        //implement logic
    }

    bool CanSubmit()
    {
        return true;
    }
}
```

?>故意 DelegateCommand 阻止使用值类型（int、double、bool 等）。因为 ICommand 需要 object ，所以在命令绑定的 XAML 初始化期间调用 时，具有 的 T 值类型会导致意外行为 CanExecute(null) 。使用 default(T) 被考虑并被拒绝作为解决方案，因为实现者无法区分有效值和默认值。如果希望使用值类型作为参数，则必须使用 DelegateCommand<Nullable<int>> 或速记 ? 语法 （ DelegateCommand<int?> ） 使其可为 null。

# 从视图调用 DelegateCommands
视图中的控件可以通过多种方式与 ViewModel 提供的命令对象相关联。某些 WPF、Xamarin.Forms 和 UWP 控件可以通过该属性轻松数据绑定到命令对象。Command
```
<Button Command="{Binding SubmitCommand}" CommandParameter="OrderId"/>
```
还可以选择使用该属性定义命令参数。预期参数的类型在泛型声明中指定。当用户与该控件交互时，该控件将自动调用该命令，并且命令参数（如果提供）将作为参数传递给该命令的方法。在前面的示例中，单击按钮时将自动调用。此外，如果指定了委托，则该按钮将在返回 时自动禁用，如果返回 ，则启用该按钮。CommandParameterDelegateCommand<T>ExecuteSubmitCommandCanExecuteCanExecutefalsetrue

引发变更通知
ViewModel 通常需要指示命令状态的更改，以便 UI 中绑定到命令的任何控件都将更新其启用状态，以反映绑定命令的可用性。提供了几种将这些通知发送到 UI 的方法。CanExecuteDelegateCommand

RaiseCanExecuteChanged
每当需要手动更新绑定的 UI 元素的状态时，请使用该方法。例如，当属性值发生更改时，我们将调用属性的 setter 以通知 UI 状态更改。RaiseCanExecuteChangedIsEnabledRaiseCanExecuteChanged
```
private bool _isEnabled;
public bool IsEnabled
{
    get { return _isEnabled; }
    set
    {
        SetProperty(ref _isEnabled, value);
        SubmitCommand.RaiseCanExecuteChanged();
    }
}
```
ObservesProperty
如果命令应在属性值更改时发送通知，则可以使用该方法。使用该方法时，每当提供的属性的值发生更改时，都会自动调用以通知 UI 状态更改。ObservesPropertyObservesPropertyDelegateCommandRaiseCanExecuteChanged
```
public class ArticleViewModel : BindableBase
{
    private bool _isEnabled;
    public bool IsEnabled
    {
        get { return _isEnabled; }
        set { SetProperty(ref _isEnabled, value); }
    }

    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit, CanSubmit).ObservesProperty(() => IsEnabled);
    }

    void Submit()
    {
        //implement logic
    }

    bool CanSubmit()
    {
        return IsEnabled;
    }
}
```
注意
使用该方法时，可以链式注册多个属性以进行观察。例：。ObservesPropertyObservesProperty(() => IsEnabled).ObservesProperty(() => CanSave)

观察 CanExecute
如果 your 是简单属性的结果，则可以省去声明委托的需要，而是使用该方法。 不仅会在注册的属性值更改时向 UI 发送通知，而且还会使用与实际委托相同的属性。CanExecuteBooleanCanExecuteObservesCanExecuteObservesCanExecuteCanExecute
```
public class ArticleViewModel : BindableBase
{
    private bool _isEnabled;
    public bool IsEnabled
    {
        get { return _isEnabled; }
        set { SetProperty(ref _isEnabled, value); }
    }

    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit).ObservesCanExecute(() => IsEnabled);
    }

    void Submit()
    {
        //implement logic
    }
}
```
警告
不要尝试链式寄存器方法。只能观察到委托的一个属性。ObservesCanExecuteCanExcute

实现基于任务的 DelegateCommand
在当今的 / 世界中，在委托内部调用异步方法是一个非常常见的要求。每个人的第一反应是他们需要一个，但这种假设是错误的。 从本质上讲，它是同步的，并且应将 AND 委托视为事件。这意味着这是用于命令的完全有效的语法。有两种方法可以将 async 方法与 .asyncawaitExecuteAsyncCommandICommandExecuteCanExecuteasync voidDelegateCommand

选项 1：
```
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(Submit);
    }

    async void Submit()
    {
        await SomeAsyncMethod();
    }
}
```
选项 2：
```
public class ArticleViewModel
{
    public DelegateCommand SubmitCommand { get; private set; }

    public ArticleViewModel()
    {
        SubmitCommand = new DelegateCommand(async ()=> await Submit());
    }

    Task Submit()
    {
        return SomeAsyncMethod();
    }
}
```