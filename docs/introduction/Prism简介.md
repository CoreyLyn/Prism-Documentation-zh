# Prism 简介

> Prism 是一个框架，用于在 WPF、.NET MAUI、Uno 平台和 Xamarin Forms 中构建松散耦合、可维护和可测试的 XAML 应用程序。每个平台都有单独的版本，这些版本将在单独的时间表上开发。Prism 提供了一组设计模式的实现，这些模式有助于编写结构良好且可维护的 XAML 应用程序，包括 MVVM、依赖项注入、命令、EventAggregator 等。Prism 的核心功能是交叉编译的 .NET Standard 和 .NET 4.5/4.8 库中的共享代码库。那些需要特定于平台的内容在目标平台的相应库中实现。Prism 还提供了这些模式与目标平台的完美集成。例如，Prism for Xamarin Forms 允许你使用可单元测试的抽象进行导航，但该抽象位于平台概念和 API 之上进行导航，以便你可以充分利用平台本身提供的功能，但以 MVVM 方式完成。
