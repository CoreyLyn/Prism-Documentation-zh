# 使用 Prism 进行依赖注入

Prism 始终围绕依赖注入构建。这有助于构建可维护和可测试的应用，并帮助您减少或消除对静态和循环引用的依赖。在 Prism 7 之前，Prism 的依赖注入主要集中在与 Prism 一起使用的各种容器上。这导致了许多问题，包括虽然可能已经编写了文档，向您展示了如何使用一个容器执行某些操作，但它们不一定反映用于您用于应用程序的容器的适当 API。

Prism 7 引入了几个新的接口，用于抽象 Prism 对依赖注入所需的内容。正如您可能想象的那样，这有几个好处，包括：

- 展示如何在 Prism 中执行某些操作的文档将始终向您显示您需要执行的操作，而无需担心您正在使用的依赖项注入容器。
- 这大大简化了需要添加到任何容器特定包的内容。在 Prism.Forms 的情况下，这将每个特定于容器的项目减少 3 个类： `PrismApplication` 、一个实现 `IContainerExtension` 和一个扩展类，用于检索底层容器，如果您觉得需要访问它，以获得 Prism 未实现的 API 之一。

在 Prism 9 中，Prism Ioc 层已从 Prism.Core 中删除，现在独立于 Prism 发货。这使我们可以更轻松地在所有受支持的 Prism 平台（WPF、Uno 平台、.NET MAUI 等）之间共享容器实现。在 Prism 9 中还完成了其他工作，以便容器更好地与 Microsoft.Extensions.DependencyInjection 集成，并为容器范围方案提供更好的支持，其中一些方案被 Prism.Maui 广泛使用。

## 使用 Microsoft 的 IServiceCollection

Prism 9.0 已将容器实现与主 Prism 存储库分开。这使我们能够在所有平台上交付和共享容器，而无需与 Prism.Core 进行任何特定的代码耦合。在更新的 Prism 9.0 实现中，添加了对 Microsoft 的 IServiceCollection 的支持。这有助于 Prism 更好地支持 .NET MAUI 应用程序和 Uno.Extensions 使用的 IHostBuilder 方法。请务必考虑，在使用各种 Microsoft 库中的注册扩展时，这些扩展将针对 Web 应用程序进行定制。例如，如果使用 EntityFrameworkCore，则 DbContext 的默认生存期将设置为 Scoped。对于大多数 Prism 应用程序，您可能希望将其设置为瞬态，因为如果不同的 ViewModel 或服务同时访问数据库，则 Singleton 可能会导致 DbAccess 问题。请务必花一些时间评估用于注册服务的任何预生成扩展方法，以便确保服务具有适合应用程序的生存期。

## 容器

Prism 团队为 Prism IoC 抽象提供了多个 DI 容器实现。

| 容器 | 获取 | 说明 |
|:---------:|:------------:|:-----:|
| DryIoc | NuGet.org | Supported across all targets |
| Grace | Commercial Plus | |
| Microsoft | Commercial Plus | |
| Unity | NuGet.org | Legacy support for Xamarin.Forms and WPF only |

?> 虽然 DryIoc 和 Unity 容器在 NuGet.org 上可用，但它们仍受 Prism 许可证的约束。您应该拥有 Prism 的有效许可证。

## Next Steps

- 了解如何 [注册服务](xref:DependencyInjection.RegisterServices)
- 了解如何 [注册用于导航的页面（特定于 Xamarin）](xref:Platforms.XamarinForms.Navigation.Basics) ***(Xamarin Specific)***
- 了解如何 [注册特定于平台的服务（Xamarin 特定 - 旧版）](xref:DependencyInjection.IPlatformInitializer) ***(Xamarin Specific - Legacy)***
- [Microsoft.Extensions.DependencyInjection（补充）](xref:DependencyInjection.Supplement)
<!-- - Learn how to [Add a Custom Container](xref:DependencyInjection.AddCustomContainer) -->
- 在[附录](xref:DependencyInjection.Appendix)中了解有关 Prism 容器扩展和使用 Shiny 的更多信息
