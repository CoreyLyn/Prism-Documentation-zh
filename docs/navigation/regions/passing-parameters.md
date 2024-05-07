# 在导航过程中传递参数

若要在应用程序中实现所需的导航行为，通常需要在导航请求期间指定其他数据，而不仅仅是目标视图名称。 **NavigationContext** 对象提供对导航 URI 以及在其内或外部指定的任何参数的访问。可以从 **IsNavigationTarget** 、 **OnNavigatedFrom** 和 **OnNavigatedTo** 方法中访问 **NavigationContext。**

Prism 提供了 **NavigationParameters** 类来帮助指定和检索导航参数。 **NavigationParameters** 类维护一个名称/值对列表，每个参数对一个。可以使用此类将参数作为导航 URI 的一部分传递，或用于传递对象参数。

下面的代码示例演示如何向 **NavigationParameters** 实例添加单个字符串参数，以便可以将其追加到导航 URI。

```cs
Employee employee = Employees.CurrentItem as Employee;
if (employee != null)
{
    var navigationParameters = new NavigationParameters();
    navigationParameters.Add("ID", employee.Id);
    _regionManager.RequestNavigate(
        RegionNames.TabRegion,
        new Uri("EmployeeDetailsView" + navigationParameters.ToString(), UriKind.Relative)
    );
}
```

此外，还可以通过将对象参数添加到 **NavigationParameters** 实例并将其作为 **RequestNavigate** 方法的参数传递来传递对象参数。以下代码使用更简单的基于字符串的导航显示了这一点：

```cs
Employee employee = Employees.CurrentItem as Employee;
if (employee != null)
{
    var parameters = new NavigationParameters();
    parameters.Add("ID", employee.Id);
    parameters.Add("myObjectParameter", new ObjectParameter());
    regionManager.RequestNavigate(RegionNames.TabRegion, "EmployeeDetailsView", parameters);
}
```

可以使用 **NavigationContext** 对象的 **Parameters** 属性检索导航参数。此属性返回 **NavigationParameters** 类的实例，该类提供索引器属性，以允许轻松访问单个参数，而与通过查询或 **RequestNavigate** 方法传递这些参数无关。

```cs
public void OnNavigatedTo(NavigationContext navigationContext)
{
    string id = navigationContext.Parameters["ID"];
    ObjectParameter myParameter = navigationContext.Parameters["myObjectParameter"];
}
```

您还可以使用泛型以类型安全的方式检索参数：

```cs
public void OnNavigatedTo(NavigationContext navigationContext)
{
    ObjectParameter objectParameter = navigationContext.Parameters.GetValue<ObjectParameter>("myObjectParameter");
}
```
