# 确认导航

您经常会发现，在导航操作期间需要与用户进行交互，以便用户可以确认或取消它。例如，在许多应用程序中，用户可能会在输入或编辑数据的过程中尝试导航。在这些情况下，您可能希望询问用户是否要在继续离开页面之前保存或放弃已输入的数据，或者用户是否要完全取消导航操作。Prism 通过 **IConfirmNavigationRequest** 接口支持这些方案。

**IConfirmNavigationRequest** 接口派生自 **INavigationAware** 接口，并添加了 **ConfirmNavigationRequest** 方法。通过在视图或视图模型类上实现此接口，可以允许他们以一种允许他们与用户交互的方式参与导航序列，以便用户可以确认或取消导航。显示确认的一种方法是使用简单的 Windows 消息框。对于更复杂的内容，可以使用对话服务，如 [Dialog Service](xref:Dialogs.GettingStarted) 中所述。

**ConfirmNavigationRequest** 方法提供两个参数，一个是对当前导航上下文的引用（如前所述），另一个回调方法，在希望继续导航时可以调用该方法。

以下步骤总结了使用 **InteractionRequest** 对象确认导航的过程：

1. 导航操作是通过 **RequestNavigate** 调用启动的。
1. 如果当前视图的视图或视图模型实现 **IConfirmNavigation** ，则调用 **ConfirmNavigationRequest。**
1. 该视图显示确认 UI 并等待用户的响应。
1. 调用 continuation 回调以继续或取消挂起的导航操作。
1. 导航操作已完成或取消。

为了说明这一点，请查看 [22-ConfirmCancelNavigation](https://github.com/PrismLibrary/Prism-Samples-Wpf/tree/master/22-ConfirmCancelNavigation) 中的示例应用。

```cs
public class ViewAViewModel : BindableBase, IConfirmNavigationRequest
{
    public ViewAViewModel()
    {
    }

    public void ConfirmNavigationRequest(NavigationContext navigationContext, Action<bool> continuationCallback)
    {
        bool result = true;

        // this is demo code only and not suitable for production. It is generally
        // poor practice to reference your UI in the view model. Use the Prism
        // IDialogService to help with this.
        if (MessageBox.Show("Do you to navigate?", "Navigate?", MessageBoxButton.YesNo) == MessageBoxResult.No)
            result = false;

        continuationCallback(result);
    }

    public bool IsNavigationTarget(NavigationContext navigationContext)
    {
        return true;
    }

    public void OnNavigatedFrom(NavigationContext navigationContext)
    {
    }

    public void OnNavigatedTo(NavigationContext navigationContext)
    {
    }
}
```

在上面的示例中，调用 ConfirmNavigationRequest 时，会弹出一个简单的 Windows 消息框，供用户说“是”或“否”。如果用户选择“否”按钮，则导航将被取消。

所有这些都发生在 UI 线程上。但是，仍然可以调用异步方法（例如 REST API 调用）来帮助确定导航状态。在这种情况下，根据实现的不同，您可能需要存储对回调的引用，以便可以从其他位置调用它。

如果存在长时间运行的实现，则用户可能会调用其他导航操作。如果发生这种情况，则上一个导航将被取消，并且调用前一个导航的回调将不起作用，因为它不再是当前导航。
