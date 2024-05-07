# 基础区域导航

视图注入和视图发现都可以被视为有限的导航形式。视图注入是显式编程导航的一种形式，视图发现是隐式或延迟导航的一种形式。但是，在 Prism 中，区域已扩展为支持基于 URI 和可扩展导航机制的更通用的导航概念。

在区域内导航意味着将在该区域内显示新视图。要显示的视图通过 URI 标识，默认情况下，URI 引用要创建的视图的名称。可以使用 **INavigateAsync** 接口定义的 **RequestNavigate** 方法以编程方式启动导航。

>**注意:** 尽管名称如此，但 **INavigateAsync** 接口并不表示在单独的后台线程上执行的异步导航。相反， **INavigateAsync** 接口表示执行伪异步导航的能力。 **RequestNavigate** 方法可以在导航操作完成后同步返回，也可以在导航操作仍处于挂起状态时返回，例如用户需要确认导航的情况。通过允许您在导航期间指定回调和延续，Prism 提供了一种机制来启用这些方案，而无需在后台线程上导航的复杂性。

**INavigateAsync** 接口由 **Region** 类实现，允许您在该区域内启动导航。

```cs
IRegion mainRegion = ...;
mainRegion.RequestNavigate(new Uri("InboxView", UriKind.Relative));
```

您还可以使用更简单的字符串重载：

```cs
IRegion mainRegion = ...;
mainRegion.RequestNavigate("InboxView");
```

您还可以在 **RegionManager** 上调用 **RequestNavigate** 方法 , 该方法允许您指定要导航的区域的名称。此便捷方法获取对指定区域的引用，然后调用 **RequestNavigate** 方法，如前面的代码示例所示。

```cs
IRegionManager regionManager = ...;
regionManager.RequestNavigate("MainRegion", new Uri("InboxView", UriKind.Relative));
```

如上所述，您可以使用字符串重载进行导航：

```cs
IRegionManager regionManager = ...;
regionManager.RequestNavigate("MainRegion", "InboxView");
```

默认情况下，导航 URI 指定在容器中注册的视图的名称。

在导航过程中，通过容器实例化指定的视图及其相应的视图模型以及其他依赖的服务和组件。实例化视图后，将其添加到指定区域并激活。有关此内容的更多信息，请参阅 [View and View Model Participation](xref:Navigation.Regions.ViewViewModelParticipation) 。

>**注意:** 前面的描述说明了视图优先导航，其中 URI 引用视图类型的名称，因为它已注册到容器。使用视图优先导航时，依赖视图模型将创建为视图的依赖关系。另一种方法是使用视图模型优先导航，其中导航 URI 引用视图模型类型的名称，因为它已注册到容器。当视图定义为数据模板时，或者当您希望独立于视图定义导航方案时，视图模型优先导航非常有用。

**RequestNavigate** 方法还允许您指定回调方法或委托，该方法或委托将在导航完成时调用。

```cs
private void SelectedEmployeeChanged(object sender, EventArgs e)
{
    ...
    regionManager.RequestNavigate(RegionNames.TabRegion, "EmployeeDetails", NavigationCompleted);
}
private void NavigationCompleted(NavigationResult result)
{
    ...
}
```

**NavigationResult** 类定义提供有关导航操作的信息的属性。 **Result** 属性指示导航是否成功。如果导航成功，则 **Result** 属性将为 _true_. 如果导航失败（通常是由于在 **IConfirmNavigationResult.ConfirmNavigationRequest** 方法中返回 'continuationCallBack(false)' , 则 **Result** 属性将为 _false_. 如果导航由于异常而失败，则 **Result** 属性将为 _false_，并且 **Error** 属性提供对导航期间引发的任何异常的引用。 **Context** 属性提供对导航 URI 及其包含的任何参数的访问，以及对协调导航操作的导航服务的引用。
