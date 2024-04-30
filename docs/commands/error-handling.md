# 错误处理

Prism 9 为包括  [AsyncDelegateCommand](xref:Commands.AsyncDelegateCommand) 在内的所有命令引入了更好的错误处理。这为应用程序开发人员提供了几个有用的机会。

1. 避免需要将每个方法包装在 `try/catch`。
2. 提供多个处理程序，以根据遇到的异常类型提供特定逻辑。
3. 在多个命令之间共享错误处理逻辑。

```cs
new DelegateCommand(() => { })
    .Catch<NullReferenceException>(nullReferenceException => {
        // Provide specific handling for the specified Exception Type
    })
    .Catch(exception => {
        // Handle any exception thrown
    })
```
