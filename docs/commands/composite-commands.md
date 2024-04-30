# 复合命令

在许多情况下，由 ViewModel 定义的命令将绑定到关联视图中的控件，以便用户可以从视图中直接调用该命令。但是在某些情况下，您可能希望能够从应用程序 UI 的父视图中的控件对一个或多个 ViewModel 调用命令。

例如，如果应用程序允许用户同时编辑多个项目，则可能希望允许用户使用应用程序工具栏或功能区中的按钮表示的单个命令保存所有项目。在这种情况下，“全部保存”命令将调用 ViewModel 实例为每个项目实现的每个“保存”命令，如下图所示。

![SaveAll composite command](../images/composite-commands-1.png)

Prism 通过 `CompositeCommand` 类支持此方案。

该 `CompositeCommand` 类表示由多个子命令组成的命令。调用复合命令时，将依次调用其每个子命令。在需要在 UI 中将一组命令表示为单个命令或要调用多个命令来实现逻辑命令的情况下，它非常有用。

该 `CompositeCommand` 类维护子命令（ `DelegateCommand` 实例）的列表。 `CompositeCommand` 该类 `Execute` 的方法只是依次调用每个子命令上 `Execute` 的方法。该 `CanExecute` 方法类似地调用每个子命令 `CanExecute` 的方法，但如果无法执行任何子命令，则该 `CanExecute` 方法将返回 `false` 。换言之，默认情况下，只有当所有子命令都可以执行时，才能执行 `CompositeCommand` 。

?> `CompositeCommand` 可以在 Prism.Core NuGet 包中的 Prism.Commands 命名空间中找到。

<iframe height="510" src="https://www.youtube.com/embed/kssprOqdfME" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## 创建复合命令

若要创建复合命令，请实例化实例， `CompositeCommand` 然后将其公开为 `ICommand` 或者 `CompositeCommand` 属性。

```cs
    public class ApplicationCommands
    {
        private CompositeCommand _saveCommand = new CompositeCommand();
        public CompositeCommand SaveCommand
        {
            get => _saveCommand;
        }
    }
```

## 使 CompositeCommand 全局可用

通常，CompositeCommands 在整个应用程序中共享，并且需要全局可用。重要的是，当您向 注册 `CompositeCommand` 子命令时，您在整个应用程序中使用相同的 CompositeCommand 实例。这要求将 CompositeCommand 定义为应用程序中的单一实例。这可以通过使用依赖关系注入 （DI） 或将 CompositeCommand 定义为静态类来完成。

### 使用依赖注入

定义 CompositeCommands 的第一步是创建接口。

```cs
    public interface IApplicationCommands
    {
        CompositeCommand SaveCommand { get; }
    }
```

接下来，创建一个实现接口的类。

```cs
    public class ApplicationCommands : IApplicationCommands
    {
        private CompositeCommand _saveCommand = new CompositeCommand();
        public CompositeCommand SaveCommand
        {
            get =>_saveCommand;
        }
    }
```

定义 ApplicationCommands 类后，必须将其注册为容器中的单一实例。

```cs
    public partial class App : PrismApplication
    {
        protected override void RegisterTypes(IContainerRegistry containerRegistry)
        {
            containerRegistry.RegisterSingleton<IApplicationCommands, ApplicationCommands>();
        }
    }
```

接下来，在 ViewModel 构造函数中请求 `IApplicationCommands` 接口。拥有该 `ApplicationCommands` 类的实例后，现在可以将 DelegateCommands 注册到相应的 CompositeCommand。

```cs
    public DelegateCommand UpdateCommand { get; private set; }

    public TabViewModel(IApplicationCommands applicationCommands)
    {
        UpdateCommand = new DelegateCommand(Update);
        applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
    }
```

### 使用静态类

创建一个表示 CompositeCommands 的静态类

```cs
public static class ApplicationCommands
{
    public static CompositeCommand SaveCommand = new CompositeCommand();
}
```

在 ViewModel 中，将子命令关联到静态 `ApplicationCommands` 类。

```cs
    public DelegateCommand UpdateCommand { get; private set; }

    public TabViewModel()
    {
        UpdateCommand = new DelegateCommand(Update);
        ApplicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
    }
```

?> 为了提高代码的可维护性和可测试性，建议您使用依赖注入方法。

## 绑定到全局可用的命令

创建 CompositeCommands 后，现在必须将它们绑定到 UI 元素才能调用这些命令。

### 使用依赖注入

在使用依赖注入（DI）时，您必须暴露 `IApplicationCommands` 以便于在视图上进行绑定。在视图的ViewModel中，需要在构造器中请求 `IApplicationCommands` ，并将一个类型为 `IApplicationCommands` 的属性设置为该实例。

```cs
    public class MainWindowViewModel : BindableBase
    {
        private IApplicationCommands _applicationCommands;
        public IApplicationCommands ApplicationCommands
        {
            get => _applicationCommands;
            set => SetProperty(ref _applicationCommands, value);
        }

        public MainWindowViewModel(IApplicationCommands applicationCommands)
        {
            ApplicationCommands = applicationCommands;
        }
    }
```

在视图中，将按钮绑定到 `ApplicationCommands.SaveCommand` 属性。 `SaveCommand` 是在 `ApplicationCommands` 类上定义的属性。

```xml
<Button Content="Save" Command="{Binding ApplicationCommands.SaveCommand}"/>
```

<!--TODO: Remove duplicate sections -->
### 使用静态类

如果使用的是静态类方法，下面的代码示例演示如何将按钮绑定到 WPF 中的静态 `ApplicationCommands` 类。

```xml
<Button Content="Save" Command="{x:Static local:ApplicationCommands.SaveCommand}" />
```

## 注销命令

如前面的示例所示，使用该 `CompositeCommand.RegisterCommand` 方法注册子命令。但是，当您不再希望响应 CompositeCommand 或要销毁 View/ViewModel 以进行垃圾回收时，应使用该 `CompositeCommand.UnregisterCommand` 方法注销子命令。

```cs
    public void Destroy()
    {
        _applicationCommands.UnregisterCommand(UpdateCommand);
    }
```

!> 当不再需要 View/ViewModel 时，你必须从 `CompositeCommand` 中注销命令（为 GC 做好准备）。否则，您将引入内存泄漏。

## 在活动视图上执行命令

父视图级别的复合命令通常用于协调子视图级别的命令的调用方式。在某些情况下，您需要执行所有显示视图的命令，如前面描述的“全部保存”命令示例中所示。在其他情况下，您希望仅在活动视图上执行该命令。在这种情况下，复合命令将仅对被视为活动的视图执行子命令;它不会对未处于活动状态的视图执行子命令。例如，您可能希望在应用程序的工具栏上实现一个缩放命令，该命令仅导致缩放当前活动项，如下图所示。

![Executing a CompositeCommand on a single child](../images/composite-commands-2.png)

为了支持此方案，Prism 提供了 `IActiveAware` 接口。该 `IActiveAware` 接口定义一个属性，该 `IsActive` 属性在实现者处于活动状态时返回 `true` ，以及一个在活动状态更改时引发 `IsActiveChanged` 的事件。

您可以在视图或 ViewModel 上实现该 `IActiveAware` 接口。它主要用于跟踪视图的活动状态。视图是否处于活动状态由特定控件中的视图确定。例如，对于 Tab 控件，有一个适配器将当前选定选项卡中的视图设置为活动状态。

该 `DelegateCommand` 类还实现 `IActiveAware` 接口。 `CompositeCommand` 可以通过在构造函数中指定 `true` `monitorCommandActivity` 参数来配置为评估子 DelegateCommands 的活动状态（除了状态之外）。 `CanExecute` 当此参数设置为 `true` 时，在确定 `CanExecute` 方法的返回值和在 `Execute` 方法中执行子命令时， `CompositeCommand` 类将考虑每个子 DelegateCommand 的活动状态。

```cs
    public class ApplicationCommands : IApplicationCommands
    {
        private CompositeCommand _saveCommand = new CompositeCommand(true);
        public CompositeCommand SaveCommand
        {
            get => _saveCommand;
        }
    }
```

当 `monitorCommandActivity` 参数为 `true` 时，该 `CompositeCommand` 类表现出以下行为：

- `CanExecute`: 仅当所有活动命令都可以执行时才返回 `true` 。根本不考虑处于非活动状态的子命令。
- `Execute`: 执行所有活动命令。根本不考虑处于非活动状态的子命令。

通过在 ViewModel 上实现该 `IActiveAware` 接口，您将在视图变为活动或非活动时收到通知。当视图的活动状态发生更改时，可以更新子命令的活动状态。然后，当用户调用复合命令时，将调用活动子视图上的命令。

```cs
    public class TabViewModel : BindableBase, IActiveAware
    {
        private bool _isActive;
        public bool IsActive
        {
            get { return _isActive; }
            set => SetProperty(ref _isActive, OnIsActiveChanged);
        }

        public event EventHandler IsActiveChanged;

        public DelegateCommand UpdateCommand { get; private set; }

        public TabViewModel(IApplicationCommands applicationCommands)
        {
            UpdateCommand = new DelegateCommand(Update);
            applicationCommands.SaveCommand.RegisterCommand(UpdateCommand);
        }

        private void Update()
        {
            //implement logic
        }

        private void OnIsActiveChanged()
        {
            UpdateCommand.IsActive = IsActive; //set the command as active
            IsActiveChanged?.Invoke(this, new EventArgs()); //invoke the event for all listeners
        }
    }
```
