# 使用导航 Journal

**NavigationContext** 类提供对区域导航服务的访问，该服务负责协调区域内导航期间的操作顺序。它提供对进行导航的区域以及与该区域关联的导航日志的访问。区域导航服务实现 **IRegionNavigationService** ，其定义如下。

```cs
public interface IRegionNavigationService : INavigateAsync
{
    IRegion Region {get; set;}
    IRegionNavigationJournal Journal {get;}
    event EventHandler<RegionNavigationEventArgs> Navigating;
    event EventHandler<RegionNavigationEventArgs> Navigated;
    event EventHandler<RegionNavigationFailedEventArgs> NavigationFailed;
}
```

由于区域导航服务实现 **INavigateAsync** 接口，因此可以通过调用其 **RequestNavigate** 方法在父区域内启动导航。启动导航操作时，将引发 **Navigating** 事件。当区域内的导航完成时，将引发 **Navigated** 事件。如果在导航过程中遇到错误，则会引发 **NavigationFailed** 。

**Journal** 属性提供对与区域关联的导航日志的访问。导航日志实现 **IRegionNavigationJournal** 接口，其定义如下。

```cs
public interface IRegionNavigationJournal
{
    bool CanGoBack { get; }
    bool CanGoForward { get; }
    IRegionNavigationJournalEntry CurrentEntry { get; }
    INavigateAsync NavigationTarget { get; set; }
    void Clear();
    void GoBack();
    void GoForward();
    void RecordNavigation(IRegionNavigationJournalEntry entry);
}
```

在导航期间，可以通过 **OnNavigatedTo** 方法调用在视图中获取和存储对区域导航服务的引用。默认情况下，Prism 提供一个简单的基于堆栈的日志，允许您在区域内向前或向后导航。

您可以使用导航日志来允许用户从视图本身进行导航。在以下示例中，视图模型实现 **GoBack** 命令，该命令使用主机区域中的导航日志。因此，视图可以显示 **后退** 按钮，使用户能够轻松地导航回区域内的上一个视图。同样，您可以实现 **GoForward** 命令来实现向导样式的工作流。

```cs
public class EmployeeDetailsViewModel : INavigationAware
{
    ...
    private IRegionNavigationService navigationService;

    public void OnNavigatedTo(NavigationContext navigationContext)
    {
        navigationService = navigationContext.NavigationService;
    }

    public DelegateCommand<object> GoBackCommand { get; private set; }

    private void GoBack(object commandArg)
    {
        if (navigationService.Journal.CanGoBack)
        {
            navigationService.Journal.GoBack();
        }
    }

    private bool CanGoBack(object commandArg)
    {
        return navigationService.Journal.CanGoBack;
    }
}
```

如果需要在某个区域内实现特定的工作流模式，则可以为该区域实施自定义日志。

>**注意:** 导航日志只能用于由区域导航服务协调的基于区域的导航操作。如果使用视图发现或视图注入在某个区域内实现导航，则导航日志在导航过程中不会更新，并且不能用于在该区域内向前或向后导航。

## 选择退出导航 Journal

使用导航日志时，显示中间页面（如初始屏幕、加载页面或对话框）可能很有用。最好不要通过调用 **IRegionNavigationJournal.GoForward()** 或 **IRegionNavigationJournal.GoBack()** 来重新访问这些页面。此行为可以通过实现 **IJournalAware** 接口来实现。

```cs
public interface IJournalAware
{
    bool PersistInHistory();
}
```

通过在 **View** 或 **View Model** 上实现 **IJournalAware** 并从 **PersistInHistory()** 返回 **false** ，页面可以选择不添加到日志历史记录中。

```cs
public class IntermediaryPage : IJournalAware
{
    public bool PersistInHistory() => false;
}
```
