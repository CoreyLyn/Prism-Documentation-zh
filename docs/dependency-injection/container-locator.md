# 容器定位器（ContainerLocator）

ContainerLocator 是 Prism 8.0 中的新增功能。引入此功能是为了帮助 Prism 摆脱对 CommonServiceLocator 的依赖关系，并解决我们必须回退到 ServiceLocator 模式（例如在 XAML 扩展中）的许多内部问题。

ContainerLocator 还为 Prism 带来了一些额外的好处，特别是对于那些使用跨平台应用程序（如 Xamarin.Forms 或 Uno 平台）的开发人员。在这些情况下，有时可能需要在初始化 PrismApplication 之前初始化容器。一个常见的例子是使用 Shiny 的应用程序。对于此类应用，可能需要将 ServiceCollection 添加到 Prism 容器，然后将 ServiceProvider 返回给 Shiny。这允许 Prism 和 Shiny 维护单个容器，而不是拥有多个容器。

?> 对于那些赞助 [Dan Siegel](https://xam.dev/sponsor-prism-dan) 的开发人员，建议您为此使用 Prism.Magician。

## 如何使用 ContainerLocator

请注意，ContainerLocator 可以通过委托创建容器来延迟设置容器实例。在调用 ContainerLocator.Container 之前，它不会创建容器。

```csharp
var createContainerExtension = () => new DryIocContainerExtension();
ContainerLocator.SetContainerExtension(createContainerExtension);
```

!> 如果未在设置创建委托后调用 `ContainerLocator.Current` 或 `ContainerLocator.Container` ，则后续调用 `SetContainerExtension` 将覆盖初始委托。

## 高级用法

请注意，此处的信息仅供高级用户使用。这些 API 有意隐藏在智能感知中，因为它们不供公众使用。使用这些风险由您自己承担，并且仅在适当的情况下使用。

如果需要访问原始 IContainerExtension， `ContainerLocator.Current` 可以通过访问。

### 测试

虽然这并不完全是一个罕见的问题，但在进行单元测试时，通常建议您重置 ContainerLocator。这可确保释放容器，并清除容器实例以及委托以创建新实例。

```csharp
public class SomeTests : IDisposable
{
    public void Dispose()
    {
        ContainerLocator.ResetContainer();
    }
}
```

## 用法示例

在 ShinyStartup 中，您可能会遇到类似情况：

```csharp
public class MyStartup : ShinyStartup
{
    private void RegisterTypes(IContainerRegistry container)
    {
        // Your normal registrations here...
    }

    private IContainerExtension CreateContainerExtension() =>
        new DryIocContainerExtension();

    public override IServiceProvider CreateServiceProvider(IServiceCollection services)
    {
        ContainerLocator.SetContainerExtension(CreateContainerExtension);
        var container = ContainerLocator.Container;
        container.RegisterServices(services);
        RegisterTypes(container);
        return container.GetContainer();
    }
}
```

在 XAML 扩展中

```csharp
public class SomeMarkupExtension : IMarkupExtension
{
    private static readonly Lazy<IEventAggregator> _lazyEventAggregator =
        new Lazy<IEventAggregator>(() => ContainerLocator.Container.Resolve<IEventAggregator>());

    private IEventAggregator EventAggregator => _lazyEventAggregator.Value;

    public object ProvideValue(IServiceProvider provider)
    {
        // your logic here...
    }
}
```
