# 处理解析错误

?> 此功能是在 Prism 8 中引入的，如果您的应用以早期版本为目标平台，则此功能不适用

由于各种原因，会发生异常。开发人员遇到的一些常见错误是服务未注册或无效的 XAML，在解析视图时生成异常。Prism 容器扩展现在非常有意捕获任何底层容器异常并抛出 `ContainerResolutionException` 。ContainerResolutionException 的目标很简单...通过提供诊断和修复代码中问题所需的信息来缩短开发循环。

包含 `ContainerResolutionException` 许多常量消息，如 `MissingRegistration` 、 `CannotResolveAbstractType` 或 `CyclicalDependency` 。除了这些常量之外，它还公开正在解析的 ServiceName 和/或 ServiceType 的属性。

```csharp
public class ModuleA : IModule
{
    private IServiceIForgotToRegister IAmADunce { get; }

    public ModuleA(IServiceIForgotToRegister iAmADummy)
    {
        IAmADunce = iAmADummy;
    }
}
```

查看上面的代码片段，我们可以看到我有一个服务作为硬依赖项注入到 ModuleA 中。不幸的是，我忘了注册它。我们当然可以挂接到 ModuleManager 中的 LoadModuleCompleted 事件，这样我们就可以看到加载模块时会发生什么，如下所示：

```csharp
protected override void InitializeModules()
{
    var manager = Container.Resolve<IModuleManager>();
    manager.LoadModuleCompleted += LoadModuleCompleted;
    manager.Run();
}

private void LoadModuleCompleted(object sender, LoadModuleCompletedEventArgs e)
{
    LoadModuleCompleted(e.ModuleInfo, e.Error, e.IsErrorHandled);
}

protected virtual void LoadModuleCompleted(IModuleInfo moduleInfo, Exception error, bool isHandled)
{
    if (error != null)
    {
        // Do Something
    }
}
```

在此示例中，我将看到错误是 `ContainerResolutionException`，并且 ServiceType 是没有 ServiceName 的 ModuleA。但这并没有真正给我足够的信息。幸运的是， ContainerResolutionException 它也有一个 `GetErrors()` 方法，使我们能够查看类型是什么以及错误是什么：

```csharp
protected virtual void LoadModuleCompleted(IModuleInfo moduleInfo, Exception error, bool isHandled)
{
    if (error != null && error is ContainerResolutionException cre)
    {
        var errors = cre.GetErrors();
        foreach((var type, var ex) in errors)
        {
            Console.WriteLine($"Error with: {type.FullName}");
            Console.WriteLine($"{ex.GetType().Name}: {ex.Message}");
        }
    }
}
```

当我们运行它时，我们应该看到如下输出：

```text
Error with: MyProject.Services.IServiceIForgotToRegister
ContainerResolutionException: No Registration was found in the container for the specified type
```
