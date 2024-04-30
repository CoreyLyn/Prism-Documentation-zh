# 使用 Prism 注册类型

与大多数依赖注入模型类似，Prism 提供了大约 3 个服务生命周期的抽象：

1) Transient（瞬态：每次请求服务或类型时获取新实例）
2) Singleton（单例：每次请求服务或类型时获取相同的实例）
3) Scoped（作用域：在每个容器作用域上获取一个新实例，但在特定容器作用域内获取相同的实例）

?> 默认情况下，Prism 不使用scoping，除非在 Prism.Maui 中，它会在每个页面周围创建一个scope。 这用于 `INavigationService`、`IPageDialogService` 和 `IDialogService` 等服务.

对于熟悉 ASP.NET Core 的用户可能也会熟悉这 3 种基本类型的依赖项注册：Transients, Singletons和 Scoped 服务。与许多服务都围绕用户请求的 Web 环境不同，对于桌面和移动应用程序，我们处理的是单个用户。因此，我们必须决定，对于内存管理和其他业务需求，我们的服务是否最适合作为将在整个应用程序中重用的单个实例，或者我们是否在每次请求时创建一个新实例，然后允许垃圾回收器在我们完成它时释放内存。

同样重要的是要考虑到 Prism 对使用指定服务注册有硬性要求。这就是 Prism 允许注册您的页面以进行导航，然后稍后根据 Uri 段（如 `MyMasterDetailPage/NavigationPage/ViewA` .因此，任何不支持开箱即用命名服务的依赖注入容器都不能也不会由 Prism 团队正式实现。

## 注册 Transient 服务

对于那些您期望每次创建时都创建一个新实例的服务，只需调用 Register 方法并提供服务类型和实现类型，除非在某些情况下，简单地注册具体类型可能更为合适。

```cs
// Where it will be appropriate to use FooService as a concrete type
containerRegistry.Register<FooService>();

containerRegistry.Register<IBarService, BarService>();
```

## 注册 Singleton 服务

很多时候，您可能需要一个服务在整个应用程序中使用，因此每次需要该服务时都创建新实例都不是一个好主意。因此，为了提供更好的内存管理，更好的做法是将此类服务设置为可以在整个应用程序中使用的单一实例。在许多情况下，您可能需要在应用程序的整个生命周期中保持其状态的服务。对于这两种情况中的任何一种，将服务注册为单一实例都更有意义。

?> 单例服务实际上并未创建，因此在应用程序首次解析服务之前不会开始使用内存。

```cs
// Where it will be appropriate to use FooService as a concrete type
containerRegistry.RegisterSingleton<FooService>();

containerRegistry.RegisterSingleton<IBarService, BarService>();
```

### 注册服务实例

虽然很多时候您希望通过简单地提供 Service 和 Implementation 类型来注册 Singleton，但有时您可能希望新建服务实例并为给定服务提供它，或者您可能希望从插件（如 MonkeyCache）注册当前实例，如下所示：

```cs
containerRegistry.RegisterInstance<IFoo>(new FooImplementation());

// Sample of using James Montemagno's Monkey Cache
Barrel.ApplicationId = "your_unique_name_here";
containerRegistry.RegisterInstance<IBarrel>(Barrel.Current);
```

## 检查服务是否已注册

很多时候，特别是在编写 Prism 模块或插件时，您可能希望检查服务是否已注册，然后根据它是否已注册执行某些操作。

?> 使用 Prism 模块时，如果对给定服务有硬依赖性，则应将其注入构造函数中，以便在初始化模块时在缺少服务类型时生成异常。仅当您的意图是注册默认实现时，才应用于 `IsRegistered` 检查它。

```cs
if (containerRegistry.IsRegistered<ISomeService>())
{
    // Do something...
}
```

## 延迟加载

如前所述，您可以注册您的服务，例如 `containerRegistry.Register<IFoo, Foo>()` 。许多开发人员可能有这样的用例：他们希望节省内存和延迟加载服务， `Func<IFoo>` 或者是 `Lazy<IFoo>` 或 。Prism 8 开箱即用地支持此功能。为此，您只需将参数添加到 ViewModel 或 Service 中，如下所示。

```cs
public class ViewAViewModel
{
    public ViewAViewModel(Func<IFoo> fooFactory, Lazy<IBar> lazyBar)
    {
    }
}
```

?> 记下服务注册类型。当您使用单例服务时，使用 `Lazy<T>` 或 `Func<T>` 解决方法通常没有意义。例如，是 `IEventAggregator` 单例。这意味着你将获得整个应用程序中使用的事件聚合器的单个实例。通过使用 `Lazy<T>` 或 `Func<T>` 最终使用更多内存，可能会对性能造成影响，而不仅仅是直接请求服务。

## 全部解析

一些开发人员可能需要为同一服务契约注册多个实现，并希望解析所有这些实现。作为一个常见的用例，Shiny使用这种模式处理其一些委托接口。这可以让您通过以小块的方式响应相同的事件来构建更模块化的代码。再次强调，在注册时您不需要做任何特别的事情。要使用这个功能，您只需要在构造函数中注入 `IEnumerable<T>`，正如这里所示。

```cs
public class SomeService
{
    public SomeService(IEnumerable<IFoo> fooCollection)
    {
    }
}
```

?> 此功能目前仅在 DryIoc 中受支持。Prism 6 发布后，使用 Unity 容器的用户可能会使用此功能。
