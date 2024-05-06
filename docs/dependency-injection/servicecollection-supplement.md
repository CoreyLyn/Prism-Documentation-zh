# 依赖注入 - 补充

您可能已经意识到，依赖注入是 .NET MAUI 中的一等公民。正如您对Microsoft产品所期望的那样，他们采用了Microsoft.Extensions.DependencyInjection包。 然而，从Prism 7.0开始，Prism继续依赖于Prism IoC抽象，使用 `IContainerRegistry` 和 `IContainerProvider`.

## IServiceCollection 扩展

尽管您可能希望使用 Microsoft.Extensions.DependencyInjection 以外的容器，但很多时候您可能希望利用使用 Microsoft.Extensions.DependencyInjection 的包提供的注册扩展。在 Prism 9.0 中，我们添加了对 Microsoft.Extensions.DependencyInjection.Abstractions 的核心引用，使我们能够确保所有容器实现都可以本机支持这些扩展。

## F.A.Q.

问题： 是否支持 EntityFrameworkCore 或者 the Microsoft.Extensions.Http HttpClientFactory, 又或者 Microsoft..... ?

答： 简短的回答是也许。这些扩展可能工作得很好，可能需要一些调整，或者你可能想去 GitHub，复制注册扩展并修改它以在你的项目中为你工作。请记住，为注册这些服务而编写的扩展是围绕为 AspNetCore 开发的开发人员构建的。这是一个与我们正在构建的环境截然不同的环境。在某些情况下，例如 Entity Framework Core，您可能会发现需要更改服务的默认生存期。例如，EntityFrameworkCore 会将 DbContext 注册为作用域服务。这在 WebAPI 中非常有意义，但是，如果正在生成 Blazor 应用程序，可能会发现自己遇到多个错误，因为各种组件正在尝试同时使用 DbContext 的同一实例。如果您使用的是 Prism 区域，您可能会发现自己遇到了同样的问题，在这种情况下，您需要将默认生存期更改为瞬态。你尝试拉入的每个库都是围绕 Microsoft.Extensions.DependencyInjection 构建的，可能会也可能不会出现怪癖，因为它们最初不是在理解在移动应用中运行的情况下构建的。

问题： 什么是作用域服务？这在 Prism.Maui 应用程序中是什么意思?

答：作用域服务可能是一个高级主题。简言之，它们为您提供了瞬态和单一实例之间的中间地带，其中服务在应用程序的整个生命周期中解析多个实例，同时为作用域中的依赖项提供相同的确切实例。对于 Prism.Maui 应用程序的上下文，作用域服务解析给定页面范围内的相同服务实例，例如 INavigationService。这意味着，无论 INavigationService 是注入到 Page 本身、ViewModel、您依赖的某些服务，甚至是 Page 生命周期后期动态创建的区域的 ViewModel 中，您始终能够访问 INavigationService 的同一实例。但是，由于下一页的范围不同，因此 INavigationService 的实例也会有所不同。您可以以相同的方式在应用程序中使用作用域内的服务。

## 延伸阅读

如果您对依赖注入有其他疑问，请务必查看完整的依赖注入主题

- [依赖注入（Dependency Injection）](xref:DependencyInjection.GettingStarted)