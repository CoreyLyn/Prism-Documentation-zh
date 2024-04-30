# 使用 ViewModelLocator

用于 `ViewModelLocator` 使用标准命名约定将 `DataContext` 视图连接到 ViewModel 的实例。

Prism `ViewModelLocator` 具有附加 `AutoWireViewModel` 属性，当设置为 `true` 调用 `ViewModelLocationProvider` 类中 `AutoWireViewModelChanged` 的方法以解析视图的 ViewModel，然后将视图的数据上下文设置为该 ViewModel 的实例时。默认情况下，此行为处于启用状态：如果您不希望视图出现此行为，则需要选择退出。

> 对于 **WPF**, 这只是使用 **区域导航** 和 ```IDialogService```. 如果您使用的是 **视图注入**, 您的视图将需要选择加入。

使用附加的 `AutoWireViewModel` 属性如下所示. 将值设置为 ```False``` 选择退出， 置为 ```True``` 显式选择加入.

```xml
<Window x:Class="Demo.Views.MainWindow"
    ...
    xmlns:prism="http://prismlibrary.com/"
    prism:ViewModelLocator.AutoWireViewModel="False">
```

若要查找 ViewModel， `ViewModelLocationProvider` 首先尝试从 `ViewModelLocationProvider.Register` 该方法可能已注册的任何映射中解析 ViewModel (请参阅 [自定义 ViewModel 注册](#custom-viewmodel-registrations))。 如果无法使用此方法解析 ViewModel，则 `ViewModelLocationProvider` 回退到基于约定的方法来解析正确的 ViewModel 类型。

本约定假定：

- ViewModel 与视图类型位于同一程序集中
- ViewModel 位于 `.ViewModels` 子命名空间中
- 视图位于 `.Views` 子命名空间中
- ViewModel 名称与视图名称相对应，并以 "ViewModel." 结尾

?> `ViewModelLocationProvider` 可以在 **Prism.Core** NuGet 包的 `Prism.Mvvm` 命名空间中找到。`ViewModelLocator` 可以在特定于平台的(**Prism.WPF**, **Prism.Forms**) NuGet包的 `Prism.Mvvm` 命名空间中找到。

?> 使用 Xamarin.Forms 进行开发时，ViewModelLocator 是必需的，并自动应用于每个视图，因为它负责向 ViewModel 提供正确的 `INavigationService` 实例。开发 Xamarin.Forms 应用时， `ViewModelLocator` 只能选择退出。

<iframe height="510" src="https://www.youtube.com/embed/I_3LxBdvJi4" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## 更改命名约定

如果应用程序不遵循 `ViewModelLocator` 默认命名约定，则可以更改约定以满足应用程序的要求。该 `ViewModelLocationProvider` 类提供了一个名为的 `SetDefaultViewTypeToViewModelTypeResolver` 静态方法，该方法可用于提供您自己的约定，用于将视图关联到视图模型。

若要更改 `ViewModelLocator` 命名约定，请重写 `App.xaml.cs` 类中 `ConfigureViewModelLocator` 的方法。然后在 `ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver` 方法中提供自定义命名约定逻辑。

```cs
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();

    ViewModelLocationProvider.SetDefaultViewTypeToViewModelTypeResolver((viewType) =>
    {
        var viewName = viewType.FullName.Replace(".ViewModels.", ".CustomNamespace.");
        var viewAssemblyName = viewType.GetTypeInfo().Assembly.FullName;
        var viewModelName = $"{viewName}ViewModel, {viewAssemblyName}";
        return Type.GetType(viewModelName);
    });
}
```

<iframe height="510" src="https://www.youtube.com/embed/o4ibaOFvfww" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## 自定义 ViewModel 注册

在某些情况下，你的应用可能遵循 `ViewModelLocator` 默认命名约定，但你有许多 ViewModel 不遵循该约定。您可以直接使用 `ViewModelLocator` 的 `ViewModelLocationProvider.Register` 方法将 ViewModel 的映射注册到特定视图，而不是尝试自定义命名约定逻辑以有条件地满足所有命名要求。

下面的示例演示在名为 `MainWindow` 的视图和名为 `CustomViewModel` 的 ViewModel 之间创建映射的各种方法。

**Type / Type**

```cs
ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), typeof(CustomViewModel));
```

**Type / Factory**

```cs
ViewModelLocationProvider.Register(typeof(MainWindow).ToString(), () => Container.Resolve<CustomViewModel>());
```

**Generic Factory**

```cs
ViewModelLocationProvider.Register<MainWindow>(() => Container.Resolve<CustomViewModel>());
```

**Generic Type**

```cs
ViewModelLocationProvider.Register<MainWindow, CustomViewModel>();
```

?> 直接向 `ViewModelLocator` 注册 ViewModel 比依赖默认命名约定更快。这是因为命名约定要求使用反射，而自定义映射则直接向 `ViewModelLocator` .

!> 该 `viewTypeName` 参数必须是视图的 Type (`Type.ToString()`) 的完全限定名称。否则，映射将失败。

<iframe height="510" src="https://www.youtube.com/embed/phMc4OuKs58" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## 控制 ViewModel 的解析方式

默认情况下， `ViewModelLocator` 将使用您选择的 DI 容器来创建 Prism 应用程序来解析 ViewModel。但是，如果您需要自定义 ViewModel 的解析方式或完全更改解析器，则可以使用该 `ViewModelLocationProvider.SetDefaultViewModelFactory` 方法实现此目的。

此示例演示如何更改用于解析 ViewModel 实例的容器。

```cs
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();

    ViewModelLocationProvider.SetDefaultViewModelFactory((viewModelType) =>
    {
        return MyAwesomeNewContainer.Resolve(viewModelType);
    });
}
```

下面是一个示例，说明如何检查为其创建 ViewModel 的视图类型，以及如何执行逻辑来控制 ViewModel 的创建方式。

```cs
protected override void ConfigureViewModelLocator()
{
    base.ConfigureViewModelLocator();

    ViewModelLocationProvider.SetDefaultViewModelFactory((view, viewModelType) =>
    {
        switch (view)
        {
            case Window window:
                //your logic
                break;
            case UserControl userControl:
                //your logic
                break;
        }

        return MyAwesomeNewContainer.Resolve(someNewType);
    });
}
```
