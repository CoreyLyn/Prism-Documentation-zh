# 使用 IPlatformInitializer

?> 以下文档仅与 Prism for Xamarin.Forms 相关。 `IPlatformInitializer` 不被任何其他平台使用。

使用 Xamarin.Forms，你可能已经阅读了如何在平台特定代码中为实现类型添加 Dependency 属性，然后使用 Xamarin.Forms DependencyService 解析它。这被认为是一个主要的反模式，在使用适当的依赖注入容器时应避免。正是出于这个原因，Prism 从 Prism 7.0 开始放弃了对使用 DependencyService 的所有支持。从 Prism 6.3 开始， `IPlatformInitializer` 引入了 Prism 6.3。这使您可以轻松地向 Prism 的容器注册类型。

<iframe height="510" src="https://www.youtube.com/embed/qMzTAOOgY8c" allow="autoplay; encrypted-media" allowfullscreen></iframe>

<hr />

这是一个非常 `IPlatformInitializer` 简单的接口，它只包含一个 `RegisterTypes` 。 `IPlatformInitializer` 然后，可以在创建应用程序时将其传递到 PrismApplication 中。对于 iOS，您可能只需使用以下代码：

```cs
public partial class AppDelegate : FormsApplicationDelegate, IPlatformInitializer
{
    public override bool FinishedLaunching(UIApplication app, NSDictionary options)
    {
        global::Xamarin.Forms.Forms.Init();
        LoadApplication(new App(this));

        return base.FinishedLaunching(app, options);
    }

    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
        containerRegistry.Register<ITextToSpeech, TextToSpeech>();
    }
}
```

请记住，您需要确保您的 App 包含正确的构造函数重载，如下所示：

```cs
public class App : PrismApplication
{
    public App(IPlatformInitializer initializer)
        : base(initializer)
    {
    }
}
```
