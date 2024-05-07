# 导航到现有视图

通常，在导航过程中重用、更新或激活应用程序中的视图更合适，而不是用新视图替换。当您导航到相同类型的视图，但需要向用户显示不同的信息或状态时，或者当相应的视图在 UI 中已可用但需要激活（即，选择或设置为最顶端）时，通常会出现这种情况。

对于第一种方案的示例，假设您的应用程序允许用户使用 **EditCustomer** 视图编辑客户记录，并且用户当前正在使用该视图编辑客户 ID 123。如果客户决定编辑客户 ID 456 的客户记录，则用户只需导航到 **EditCustomer** 视图并输入新的客户 ID。然后， **EditCustomer** 视图可以检索新客户的数据并相应地更新其 UI。

第二种方案的一个示例是，应用程序允许用户一次编辑多个客户记录。在这种情况下，应用程序在选项卡控件中显示多个 **EditCustomer** 视图实例，例如，一个用于客户 ID 123，另一个用于客户 ID 456。当用户导航到 **EditCustomer** 视图并输入客户 ID 456 时，将激活相应的视图（即，将选择其相应的选项卡）。如果用户导航到 **EditCustomer** 视图并输入客户 ID 789，则将创建一个新实例并显示在选项卡控件中。

出于各种原因，导航到现有视图的功能非常有用。更新现有视图通常比将其替换为相同类型的新实例更有效。同样，激活现有视图（而不是创建重复视图）可提供更一致的用户体验。此外，无需太多自定义代码即可无缝处理这些情况的能力意味着应用程序更易于开发和维护。

Prism 通过 **INavigationAware** 接口上的 **IsNavigationTarget** 方法支持前面所述的两种方案。在对区域中与目标视图类型相同的所有视图进行导航时，将调用此方法。在前面的示例中，视图的目标类型是 **EditCustomer** 视图，因此将在区域中当前的所有现有 **EditCustomer** 视图实例上调用 **IsNavigationTarget** 方法。Prism 从视图 URI 确定目标类型，它假定视图 URI 是目标类型的短类型名称。

> **注意:** 要使 Prism 确定目标视图的类型，导航 URI 中的视图名称应与实际目标类型的短类型名称相同。例如，如果视图由 **MyApp.Views.EmployeeDetailsView** 类实现，则导航 URI 中指定的视图名称应为 **EmployeeDetailsView** 。这是 Prism 提供的默认行为。可以通过实现自定义内容加载程序类来自定义此行为：通过实现 **IRegionNavigationContentLoader** 接口或从 **RegionNavigationContentLoader** 类派生来执行此操作。

**IsNavigationTarget** 方法的实现可以使用 **NavigationContext** 参数来确定它是否可以处理导航请求。 **NavigationContext** 对象提供对导航 URI 和导航参数的访问。在前面的示例中，此方法在 **EditCustomer** 视图模型中的实现将当前客户 ID 与导航请求中指定的 ID 进行比较，如果它们匹配，则返回 **true** 。

```cs
public class EmployeeDetailsViewModel : BindableBase, INavigationAware
{
    public bool IsNavigationTarget(NavigationContext navigationContext)
    {
        string id = navigationContext.Parameters["ID"];
        return _currentCustomer.Id.Equals(id);
    }

    public void OnNavigatedTo(NavigationContext navigationContext) { }
    public void OnNavigatedFrom(NavigationContext navigationContext) { }
}
```

如果 **IsNavigationTarget** 方法始终返回 **true** ，则无论导航参数如何，都将始终重复使用该视图实例。这样可以确保在特定区域中仅显示一个特定类型的视图。
