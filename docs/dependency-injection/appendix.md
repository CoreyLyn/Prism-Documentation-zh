# Prism 容器扩展

Prism 的开箱即用注册方法涵盖了大多数依赖项注入方案。事实上，除了注册 Pages for Navigation 之外，还有很多项目只需要简单地将服务注册为 Transient 或 Singleton 之外。但是，有时您可能需要执行以下操作：

- 使用工厂方法注册类型
- 为多个服务注册单个实现类型
  - 如果您将 Prism 注册为 Transient，则可以简单地使用 Prism 完成此操作，但是容器扩展允许您使用一行代码来执行此操作
  - 您可能还需要将服务设置为单一实例，并且无论解析哪个接口都使用同一实例。

除此之外，您可能还希望利用容器扩展中 Prism.Forms 的扩展版本，因为它默认提供了更多内置的调试钩子，以便为应用程序中遇到的未捕获异常提供更好的上下文。有关详细信息，请务必查看 [Prism.Container.Extensions](https://github.com/dansiegel/Prism.Container.Extensions) 存储库.

## 支持 Shiny Lib

毫无疑问，Xamarin 开发人员最好的新库之一是 Allan Ritchie 的 Shiny 库。这提供了许多功能，包括处理设置、确定您是否连接到网络、后台任务、蓝牙等。Shiny 从一开始就考虑到了依赖注入。这有很多好处，包括它使用基于接口的方法，允许您模拟 Shiny 中的任何服务，您可以将其注入 ViewModel。

Shiny 的一个缺点是，它需要在 Xamarin.Forms 或 Prism 有机会初始化之前进行初始化。这从根本上改变了依赖注入的事实来源。为了正确组合 Prism 和 Shiny，需要正确提供 DI 容器作为 Shiny 将使用的 IServiceProvider。执行此操作的最简单方法是使用 [Prism.Container.Extensions](https://github.com/dansiegel/Prism.Container.Extensions) 中的容器实现以及同一存储库中的 [Shiny.Prism](https://www.nuget.org/packages/Shiny.Prism) NuGet。

?> 如果使用扩展表单包（Prism.DryIoc.Forms.Extended 或 Prism.Unity.Forms.Extended），则不应使用 Prism 中的容器特定包（Prism.DryIoc.Forms 或 Prism.Unity.Forms），因为这些包中的 PrismApplication 已配置为使用正确的 PrismContainerExtension。

您可以选择将 Prism.Forms 直接用于其中一个容器扩展包。如果这样做，您将需要更新您的应用，如下所示：

```cs
public partial class App : PrismApplicationBase
{
    protected override IContainerExtension CreateContainerExtension() =>
        PrismContainerExtension.Current;
}
```

使用 Shiny.Prism 中的 Startup 基类，只需提供容器扩展，如下所示：

```cs
public class Startup : PrismStartup
{
    public Startup()
        : base(PrismContainerExtension.Current)
    {
    }

    protected override void ConfigureServices(IServiceCollection services)
    {
        // Register any types you need to with Shiny
    }
}
```
