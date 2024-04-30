# BindableBase

就像您对任何 MVVM 库的期望一样，Prism 提供了一个基类实现 `INotifyPropertyChanged` 。虽然了解如何使用它很重要，但同样重要的是要注意 Prism 中的功能，例如响应生命周期事件、导航等......都是接口驱动的。这意味着，虽然 Prism 提供了 `BindableBase` 作为基本实现 `INotifyPropertyChanged` 以更好地帮助您，但 Prism 也没有严格要求您使用它。这意味着您可以为 ViewModel 使用所需的任何基类，但根本不包含基类（尽管通常不建议这样做）。

## 创建属性

在继承的类 `BindableBase` 中使用的属性必须通知 UI 更改，应使用 `SetProperty` 该方法来设置更改，并且应同时具有公共属性和私有支持字段。结果是这样的：

```cs
public class ViewAViewModel : BindableBase
{
    private string _message;
    public string Message
    {
        get => _message;
        set => SetProperty(ref _message, value);
    }
}
```

### 为什么使用 SetProperty

您可能想知道，为什么要使用 `SetProperty` ？毕竟，你不能自己调用RaisePropertyChanged吗？简短的回答是你可以。但是，这通常是不建议的，因为您将丢失内置的 EqualityComparer，这有助于确保如果多次调用具有相同值 `INotifyPropertyChanged` 的 setter，则仅在事件第一次更改时触发 `PropertyChanged` 事件。

```cs
public class ViewAViewModel : BindableBase
{
    private string _message;
    public string Message
    {
        get => _message;
        set
        {
            // Don't do this!
            _message = value;
            RaisePropertyChanged();
        }
    }
}
```

?> 当我们在生产环境中查看代码时，我们通常会遇到类似上述示例的代码。此代码从根本上存在缺陷，过于冗长，将导致为属性引发不必要的 PropertyChanged 事件。应始终以 SetProperty 为基础编写代码流。

### 在 PropertyChanges 上执行委托

有时，您可能希望在属性更改时提供回调。一个这样的示例可能是您正在实现 `IActiveAware` ，并且您希望提供一个仅在 IsActive 为 true 时执行的方法，并在 IsActive 为 false 时执行另一个方法。

```cs
public abstract class ViewModelBase : BindableBase, IActiveAware
{
    private bool _isActive;
    public bool IsActive
    {
        get => _isActive;
        set => SetProperty(ref _isActive, value, () => {
            if (value)
                OnIsActive();
            else
                OnIsNotActive();
        });
    }

    protected virtual void OnIsActive() { }
    protected virtual void OnIsNotActive() { }
}
```

