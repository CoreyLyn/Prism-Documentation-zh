# View 和 View Model 参与导航

通常，应用程序中的视图和视图模型将希望参与导航。 **INavigationAware** 接口支持此功能。您可以在视图或（更常见的）视图模型上实现此接口。通过实现此接口，您的视图或视图模型可以选择参与导航过程。

>**注意:** 在下面的描述中，尽管在视图之间的导航过程中引用了对此接口的调用，但应注意，无论 **INavigationAware** 接口是由视图还是视图模型实现，都将在导航过程中调用该接口。
在导航过程中，Prism 会检查视图是否实现了 **INavigationAware** 接口; 如果是这样，它会在导航过程中调用所需的方法。Prism 还会检查设置为视图的 **DataContext** 的对象是否实现了此接口; 如果是这样，它会在导航过程中调用所需的方法。

此接口允许视图或视图模型参与导航操作。 **INavigationAware** 接口定义了三个方法。

```cs
public interface INavigationAware
{
    bool IsNavigationTarget(NavigationContext navigationContext);
    void OnNavigatedTo(NavigationContext navigationContext);
    void OnNavigatedFrom(NavigationContext navigationContext);
}
```

**IsNavigationTarget** 方法允许现有（显示的）视图或视图模型指示它是否能够处理导航请求。在可以重用现有视图来处理导航操作或导航到已存在的视图时，这非常有用。例如，可以更新显示客户信息的视图以显示其他客户的信息。有关使用此方法的详细信息，请参阅 [Navigating to Existing Views](xref:Navigation.Regions.NavigationExistingViews).

在导航操作期间调用 **OnNavigatedFrom** 和 **OnNavigatedTo** 方法。 如果区域中的当前活动视图实现此接口（或其视图模型），则在进行导航之前调用其 **OnNavigatedFrom** 方法。 如果区域中的当前活动视图实现此接口（或其视图模型），则在进行导航之前调用其 **OnNavigatedFrom** 方法。 **OnNavigatedFrom** 方法允许上一个视图保存任何状态，或准备将其停用或从 UI 中删除，例如，保存用户对 Web 服务或数据库所做的任何更改。

如果新创建的视图实现此接口（或其视图模型），则在导航完成后调用其 **OnNavigatedTo** 方法。 **OnNavigatedTo** 方法允许新显示的视图初始化自身，可能使用导航 URI 上传递给它的任何参数。有关传递参数的详细信息，请参阅 [Passing Parameters During Navigation](xref:Navigation.Regions.PassingParameters).

实例化、初始化新视图并将其添加到目标区域后，它将成为活动视图，并停用上一个视图。有时，您会希望从该区域中删除已停用的视图。Prism 提供 **IRegionMemberLifetime** 接口，该接口允许您指定是从区域中删除已停用的视图还是仅标记为已停用，从而控制区域内视图的生存期。

```cs
public class EmployeeDetailsViewModel : IRegionMemberLifetime
{
    public bool KeepAlive
    {
        get { return true; }
    }
}
```

**IRegionMemberLifetime** 接口定义单个只读属性 **KeepAlive** 。如果此属性返回 **false** ，则在停用视图时，该视图将从区域中删除。由于该区域不再具有对视图的引用，因此它有资格进行垃圾回收（除非应用程序中的某个其他组件保留了对视图的引用）。您可以在视图或视图模型类上实现此接口。尽管 **IRegionMemberLifetime** 接口主要用于允许您在激活和停用期间管理区域内视图的生存期，但在目标区域中激活新视图后的导航过程中，也会考虑 **KeepAlive** 属性。

>**注意:** 可以显示多个视图的区域（例如使用 **ItemsControl** 或 **TabControl** 的区域）将同时显示非活动视图和活动视图。从这些类型的区域中删除非活动视图将导致该视图从 UI 中删除。