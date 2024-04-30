# INavigationParameters

导航参数是一种在导航事件期间传递状态、选项或其他值的方法。这包括 [基于页面的导航](xref:Navigation.INavigationParameters) 以及 [基于区域的导航](xref:Navigation.Regions.GettingStarted)。 导航参数可以完全由导航 Uri 中的查询字符串组成，也可以由 `NavigationParameters` .它甚至可以将两者合并，允许您组合查询字符串参数和来自 `NavigationParameters` .这将由 Prism 在导航服务中自动为您完成。

在 Prism 9.0 中，导航参数和界面在所有导航范式和平台上完全从 Prism.Core 共享。

## 创建导航参数

```cs
new NavigationParameters
{
    { "Title", "Hello World" },
    { "StarshipLaunchAttempt", new DateTime(2023, 4, 20) }
}
```

正如您在创建 NavigationParameters 实例时会注意到的那样，您可以添加各种类型的值。

```cs
new NavigationParameters
{
    { "SelectedColors", Colors.Blue },
    { "SelectedColors", Colors.Gray }
}
```

虽然乍一看似乎 `INavigationParameters` 只是一个 `IDictionary<string, object>` ，但实际上它是一个 `IEnumerable<KeyValuePair<string, object>>` .这意味着，您可以使用单个键重载键，将多个值添加到 NavigationParameters。

## 访问导航参数

根据您需要从导航参数中获取的内容，您可能需要调用以下 API 之一。

### Getting a Single Value

若要从导航参数中访问单个值，应使用如下 `GetValue` 方法：

```cs
Title = parameters.GetValue<string>("Title");
```

### 如果键存在，则获取值

若要仅在键存在时访问值，可以使用如下 `TryGetValue` 方法：

```cs
if (parameters.TryGetValue<string>("Title", out var title))
{
    Title = title;
}
```

### 获取多个值

若要访问多个值，可以使用该 `GetValues` 方法。如果未提供任何值，这将返回一个空列表。

```cs
var colors = parameters.GetValues<Color>("SelectedColors");
```
