# Prism 简介

> Prism 是一个用于构建松耦合、可维护且可测试的 XAML 应用程序的框架，适用于 WPF、.NET MAUI、Uno平台和 Xamarin Forms。每个平台都有独立的发布版本，并且会在不同的时间线上独立开发。Prism 提供了一组设计模式的实现，这些模式在撰写结构良好、易于维护的 XAML 应用程序中非常有用，包括 MVVM、依赖注入、命令、EventAggregator 等。Prism 的核心功能是基于跨编译的 .NET Standard 和 .NET 4.5/4.8 库的共享代码库。需要针对特定平台的内容将在目标平台的各个库中实现。Prism 还针对目标平台与这些模式的集成提供了大量支持。例如，Prism for Xamarin Forms 允许您使用可进行单元测试的导航抽象，但该层叠在平台概念和API上以实现导航，这样您就可以充分利用平台本身的导航提供的功能，但这一切都以 MVVM 的方式进行。

> Prism 9代表了应用程序开发人员的重大飞跃，大量的焦点集中在统一所有平台的 API上。这将为开发人员解锁许多可能性，允许他们将代码从旧的应用程序进行迭代或从一个应用程序开发平台过渡到另一个，最大化代码重用并消除开发成本。

# 许可证
请注意，Prism 9的许可证已经改变。为了帮助确保Prism继续成为一个可持续的项目，Prism 9和未来版本的Prism将在社区/商业许可证下发行。
```
Prism can be licensed either under the Prism Community License or the Prism Commercial license.

To be qualified for the Prism Community License you must have an annual gross revenue of less than one (1) million U.S. dollars ($1,000,000.00 USD) per year or have never received more than $3 million USD in capital from an outside source, such as private equity or venture capital, and agree to be bound by Prism's terms and conditions.

Customers who do not qualify for the community license can visit the Prism Library website (https://prismlibrary.com/) for commercial licensing options.

Under no circumstances can you use this product without (1) either a Community License or a Commercial License and (2) without agreeing and abiding by Prism's license containing all terms and conditions. 

The Prism license that contains the terms and conditions can be found at
https://cdn.prismlibrary.com/downloads/prism_license.pdf
```

# 商业增值许可证
商业增值许可证为开发人员提供了一系列额外的软件包。撰写文档时，这将包括：
- Prism.Plugin.Popups（.NET MAUI）
- Prism.Plugin.Essentials - 结合了几个Shiny和.NET MAUI Essentials API的交叉产品，在所有Prism平台上进行了抽象和支持，让您在开发任何平台时都可以使用相同的ViewModel来使用这些API。
- Prism.Magician - 一系列的Roslyn分析器、源代码生成器和IL织入工具，帮助您减少编码量并更快地发现代码中的问题
- 支持额外的容器
    - Microsoft.Extensions.DependencyInjection
    - Grace Ioc

此外，商业增值许可证持有者还可以访问一个私人Discord群组，在那里他们可以提问、互相帮助以及直接从Prism团队获得帮助。