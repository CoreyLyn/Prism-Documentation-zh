# 异步命令

重要的是要考虑到命令实际上是一个 EventHandler，因此，当命令仍在执行时，可以在命令执行时多次调用命令，尤其是在异步任务的上下文中。Prism 9.0 引入了 `AsyncDelegateCommand` 和 `AsyncDelegateCommand<T>`。

```cs
public interface IAsyncCommand : ICommand
{
    Task ExecuteAsync(object? parameter);

    Task ExecuteAsync(object? parameter, CancellationToken cancellationToken);
}
```

虽然没有平台明确支持 AsyncCommand 的概念，并且 Async Command 的任何实现都是特定于实现库的。但是，这确实使你能够使用自己的 Command 实现创建自定义控件，该实现确实支持 `IAsyncCommand` .这样做的一个好处 `AsyncDelegateCommand` 是，它支持可能接受也可能不接受 CancellationToken 的委托。

## 并行执行

默认情况下，不允许 `AsyncDelegateCommand` 并行执行。要启用并行执行，必须调用 `EnableParallelExecution` 。作为此默认行为 `CanExecute` 的一部分，在执行命令时将自动返回 `false`，而不管您为 `CanExecute` 委托提供的任何其他自定义逻辑如何。

```cs
new AsyncDelegateCommand(async () => Task.CompletedTask)
    .EnableParallelExecution()
```

## 配置 CancellationTokenSource

有两种方法可以为命令配置 CancellationTokenSource。

1. 提供 `TimeSpan` 可为 Async 命令提供默认超时。
2. 提供 `Func<CancellationToken>` 以提供 `CancellationToken` Command来使用.
