# IDialogAware

该 `IDialogAware` 界面在 Prism 9 中进行了一些更新，通过提供能够适应您需求的更强大的 API，帮助您编写更好地满足您需求的代码。

```cs
public interface IDialogAware
{
    bool CanCloseDialog();
    void OnDialogClosed();
    void OnDialogOpened(IDialogParameters parameters)
    DialogCloseListener RequestClose { get; }
}
```

## Can Close

该 `IDialogAware` 接口提供了方法 `CanCloseDialog() `。这允许您提供任何自定义逻辑，以确定是否应允许关闭对话框：

```cs
public class MyDialogViewModel : IDialogAware
{
    private string? _name;
    public string? Name
    {
        get => _name;
        set => SetProperty(ref _name, value)
    }

    public bool CanCloseDialog() => !string.IsNullOrEmpty(Name);
}
```

## DialogCloseListener

DialogCloseListener 是 Prism 9 中的新增功能，它取代了原始 API 中的事件。DialogCloseListener 为您提供了更大的灵活性，并且是 Dialog Service 增强 API 的一部分。

?> RequestClose 属性应按如下所示实现。此属性由 DialogService 本身设置，不应由代码设置。

```cs
public class MyDialogViewModel : IDialogAware
{
    public DialogCloseListener RequestClose { get; }
}
```

### 使用 DialogCloseListener

DialogCloseListener 的优点之一是，它在调用它时具有更大的灵活性。

```cs
private void OnMyCommandExecuted()
{
    // Option 1.
    RequestClose.Invoke();

    // Option 2.
    RequestClose.Invoke(new DialogParameters{ { "MyParameter", SomeValue } });

    // Option 3.
    RequestClose.Invoke(ButtonResult.OK);

    // Option 4.
    RequestClose.Invoke(new DialogParameters{ { "MyParameter", SomeValue } }, ButtonResult.OK);

    // Option 5.
    var result = new DialogResult
    {
        Parameters = new DialogParameters{ { "MyParameter", SomeValue } },
        Result = ButtonResult.OK
    };
    RequestClose.Invoke(result);
}
```

## 其他注意事项

使用 .NET MAUI 生成应用时，可能需要考虑使用弹出页。使用 Commercial Plus 许可证，您可以利用 [`Prism.Plugin.Popups` package for .NET MAUI](xref:Plugins.Popups).
