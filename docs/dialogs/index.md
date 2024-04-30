# 开始

您可能希望在应用程序中创建对话框的原因有很多。这可能是向用户显示消息，或向他们提供输入某些信息的表单等。在 Prism.Core 中，我们定义了一个中央抽象层，用于在所有 Prism 支持的平台上呈现对话。Prism 中的对话框使用本机机制来显示您的自定义视图。这使您能够创建与应用程序其余部分具有相同外观的对话框，同时继续使用 MVVM 模式。

## 变化

Prism 9.0 引入了一些更改， `IDialogService` 目的是帮助您使用满足您需求的回调代码。更改的核心是引入 DialogCallback。DialogCallback 旨在让您更灵活地响应 `IDialogResult` .这允许您提供异步或同步委托。最后，它不是明确规定您可能需要提供什么作为回调的论据，而是旨在更好地满足您的需求。

### 关闭

```cs
// Basic Callback
new DialogCallback().OnClose(() => Console.WriteLine("The Dialog Closed"));

// Callback with the Dialog Result
new DialogCallback().OnClose(result => Console.WriteLine($"The Dialog Button Result is: {result.Result}"));
```

除了上面显示的同步回调之外，每个回调都有一个处理异步回调的等效项：

```cs
// Basic Callback
new DialogCallback().OnCloseAsync(() => Task.CompletedTask);

// Callback with the Dialog Result
new DialogCallback().OnCloseAsync(result => Task.CompletedTask);
```

### 错误处理

此外，它还允许您提供一个错误处理程序，该处理程序仅在遇到异常时才会调用。

```cs
// Basic Error Callback
new DialogCallback().OnError(() => Console.WriteLine("Whoops... something bad happened!"));

// Catch All Exception Handler
new DialogCallback().OnError(exception => Console.WriteLine(exception));

// Specific Catch Handler
new DialogCallback().OnError<NullReferenceException>(nre =>
{
    Console.WriteLine("This will only be executed when the Exception is a NullReferenceException.");
    Console.WriteLine("Plus our variable is correctly typed for our handler to work with!");
});

// Specific Catch with the IDialogResult
new DialogCallback().OnError<NullReferenceException>((nre, result) =>
{
    Console.WriteLine($"Button Result: {result.Result}");
    Console.WriteLine(nre);
});
```

?> 上面的每个 `OnError` 示例也有一个等效项，该等效 `OnErrorAsync` 项也接受返回任务的委托。

## 后续步骤

- [IDialogAware 视图模型](xref:Dialogs.IDialogAware)
- [IDialogWindow](xref:Dialogs.DialogWindow) (仅限 WPF & Uno 平台)
