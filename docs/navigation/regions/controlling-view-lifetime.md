# 控制视图生存期

## IRegionMemberLifetime

实例化、初始化新视图并将其添加到目标区域后，它将成为活动视图，并停用上一个视图。有时，您会希望从该区域中删除已停用的视图。Prism 提供 **IRegionMemberLifetime** 接口，该接口允许您指定是从区域中删除已停用的视图还是仅标记为已停用，从而控制区域内视图的生存期。

```cs
public class EmployeeDetailsViewModel : BindableBase, IRegionMemberLifetime
{
    public bool KeepAlive
    {
        get { return true; }
    }
}
```

**IRegionMemberLifetime** 接口定义单个只读属性 **KeepAlive** 。如果此属性返回 **false** ，则在停用视图时，该视图将从区域中删除。由于该区域不再具有对视图的引用，因此它有资格进行垃圾回收（除非应用程序中的某个其他组件保留了对视图的引用）。您可以在视图或视图模型类上实现此接口。尽管 **IRegionMemberLifetime** 接口主要用于允许您在激活和停用期间管理区域内视图的生存期，但在目标区域中激活新视图后的导航过程中，也会考虑 **KeepAlive** 属性。

## RegionMemberLifetime属性

您可以改用属性来完成相同的操作。

```cs
[RegionMemberLifetime(KeepAlive = true)]
public class EmployeeDetailViewModel : BindableBase
{
}
```

**注意:** 可以显示多个视图的区域（例如使用 **ItemsControl** 或 **TabControl** 的区域）将同时显示非活动视图和活动视图。从这些类型的区域中删除非活动视图将导致该视图从 UI 中删除。
