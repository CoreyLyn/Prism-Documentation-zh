# 事件聚合器

Prism Library 提供了一种事件机制，用于实现应用程序中松散耦合组件之间的通信。此机制基于事件聚合器服务，允许发布者和订阅者通过事件进行通信，并且仍然没有直接相互引用。

提供 `EventAggregator` 组播发布/订阅功能。这意味着可以有多个发布者引发同一事件，并且可以有多个订阅者收听同一事件。请考虑使用 `EventAggregator` 跨模块发布事件，以及在业务逻辑代码（如控制器和演示者）之间发送消息时。

使用 Prism 库创建的事件是类型化事件。这意味着，在运行应用程序之前，您可以利用编译时类型检查来检测错误。在 Prism 库中，订阅 `EventAggregator` 者或发布者可以查找特定的 `EventBase` .事件聚合器还允许多个发布者和多个订阅者，如下图所示。

![Using the event aggregator](images/event-aggregator-1.png)

<iframe height="510" src="https://www.youtube.com/embed/xTP9_hN_3xA" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## IEventAggregator

该 `EventAggregator` 类在容器中作为服务提供，可以通过 `IEventAggregator` 接口检索。事件聚合器负责查找或构建事件，并负责在系统中保留事件的集合。

```cs
public interface IEventAggregator
{
    TEventType GetEvent<TEventType>() where TEventType : EventBase;
}
```

如果事件尚未构造，则在首次访问时 `EventAggregator` 构造事件。这样一来，发布者或订阅者就无需确定事件是否可用。

## PubSubEvent

连接发布者和订阅者的实际工作是由 `PubSubEvent` 类完成的。这是 Prism Library 中包含的 `EventBase` 类的唯一实现。此类维护订阅服务器列表并处理向订阅服务器发送的事件。

该 `PubSubEvent` 类是一个泛型类，它要求将有效负载类型定义为泛型类型。这有助于在编译时强制发布者和订阅者提供正确的方法来成功连接事件。下面的代码演示 PubSubEvent 类的部分定义。

?> `PubSubEvent` 可以在 Prism.Core NuGet 包中的 Prism.Events 命名空间中找到。

## 创建 Event

旨在 `PubSubEvent<TPayload>` 成为应用程序或模块的特定事件的基类。 `TPayLoad`是事件有效负载的类型。有效负载是在发布事件时将传递给订阅者的参数。

例如，以下代码显示 `TickerSymbolSelectedEvent` .有效负载是包含公司符号的字符串。请注意，此类的实现是空的。

```cs
public class TickerSymbolSelectedEvent : PubSubEvent<string>{}
```

?> 在复合应用程序中，事件经常在多个模块之间共享，因此它们在公共位置进行定义。通常的做法是在共享程序集（如“Core”或“Infrastructure”项目）中定义这些事件。

## 发布 Event

发布者通过从 中检索事件 `EventAggregator` 并调用 `Publish` 该方法引发事件。要访问 `EventAggregator` ，可以通过向类构造函数添加类型 `IEventAggregator` 参数来使用依赖注入。

```cs
public class MainPageViewModel
{
    IEventAggregator _eventAggregator;
    public MainPageViewModel(IEventAggregator ea)
    {
        _eventAggregator = ea;
    }
}
```

下面的代码演示如何发布 TickerSymbolSelectedEvent。

```cs
_eventAggregator.GetEvent<TickerSymbolSelectedEvent>().Publish("STOCK0");
```

## 订阅 Events

订阅者可以使用 `PubSubEvent` 类上可用的 `Subscribe` 方法重载之一来登记事件。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        ea.GetEvent<TickerSymbolSelectedEvent>().Subscribe(ShowNews);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

有几种方法可以订阅 `PubSubEvents` .使用以下条件来帮助确定哪个选项最适合您的需求：

- 如果需要能够在收到事件时更新 UI 元素，请订阅以在 UI 线程上接收事件。
- 如果需要筛选事件，请在订阅时提供筛选器委托。
- 如果对事件有性能问题，请考虑在订阅时使用强引用的委托，然后手动取消订阅 PubSubEvent。
- 如果上述情况均不适用，请使用默认订阅。

以下各节介绍这些选项。

### 在 UI 线程上订阅

通常，订阅者需要更新 UI 元素以响应事件。在 WPF 中，只有 UI 线程可以更新 UI 元素。

默认情况下，订阅者在发布者的线程上接收事件。如果发布者从 UI 线程发送事件，则订阅者可以更新 UI。但是，如果发布者的线程是后台线程，则订阅者可能无法直接更新 UI 元素。在这种情况下，订阅者需要使用 Dispatcher 类在 UI 线程上计划更新。

Prism Library `PubSubEvent` 提供的 Prism 库可以通过允许订阅者在 UI 线程上自动接收事件来提供帮助。订阅服务器在订阅期间指示这一点，如下面的代码示例所示。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        ea.GetEvent<TickerSymbolSelectedEvent>().Subscribe(ShowNews, ThreadOption.UIThread);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

以下选项可用于 `ThreadOption` ：

- `PublisherThread`: 使用此设置可在发布商的话题上接收事件。这是默认设置。
- `BackgroundThread`: 使用此设置可在 .NET Framework 线程池线程上异步接收事件。
- `UIThread`: 使用此设置在 UI 线程上接收事件。

?> `PubSubEvent` 为了在 UI 线程上发布到订阅者，`EventAggregator` 最初必须在 UI 线程上构造。

### 订阅筛选

订阅者可能不需要处理已发布事件的每个实例。在这些情况下，订阅者可以使用 filter 参数。filter 参数的类型 `System.Predicate<TPayLoad>` 是一个委托，在发布事件时执行该委托，以确定已发布事件的有效负载是否与调用订阅者回调所需的一组条件匹配。如果负载不符合指定条件，则不会执行订阅者回调。

通常，此筛选器以 lambda 表达式的形式提供，如下面的代码示例所示。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        TickerSymbolSelectedEvent tickerEvent = ea.GetEvent<TickerSymbolSelectedEvent>();
        tickerEvent.Subscribe(ShowNews, ThreadOption.UIThread, false, 
                              companySymbol => companySymbol == "STOCK0");
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

?> 该 `Subscribe` 方法返回一个类型的 `Prism.Events.SubscriptionToken` 订阅令牌，该令牌可用于稍后删除对事件的订阅。当您使用匿名委托或 lambda 表达式作为回调委托时，或者当您使用不同的筛选条件订阅同一事件处理程序时，此令牌特别有用。

?> 建议不要从回调委托中修改有效负载对象，因为多个线程可以同时访问有效负载对象。您可以将有效负载设置为不可变的，以避免并发错误。

### 使用强引用订阅

如果您在短时间内引发多个事件，并且注意到这些事件的性能问题，则可能需要订阅强大的委托引用。如果这样做，则需要在处置订阅者时手动取消订阅该事件。

默认情况下， `PubSubEvent` 维护对订阅者的处理程序和订阅筛选器的弱委托引用。这意味着 `PubSubEvent` 保留的引用不会阻止订阅者的垃圾回收。使用弱委托引用可使订阅者无需取消订阅，并允许进行适当的垃圾回收。

但是，维护此弱委托引用比相应的强引用慢。对于大多数应用程序，此性能并不明显，但如果应用程序在短时间内发布大量事件，则可能需要对 PubSubEvent 使用强引用。如果确实使用强委托引用，则订阅者应取消订阅，以便在不再使用订阅对象时对订阅对象进行适当的垃圾回收。

若要使用强引用进行订阅，请使用 `Subscribe` 方法上的 `keepSubscriberReferenceAlive` 参数，如下面的代码示例所示。

```cs
public class MainPageViewModel
{
    public MainPageViewModel(IEventAggregator ea)
    {
        bool keepSubscriberReferenceAlive = true;
        TickerSymbolSelectedEvent tickerEvent = ea.GetEvent<TickerSymbolSelectedEvent>();
        tickerEvent.Subscribe(ShowNews, ThreadOption.UIThread, keepSubscriberReferenceAlive, 
                              companySymbol => companySymbol == "STOCK0");
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

`keepSubscriberReferenceAlive` 参数类型为 `bool` ：

- 当设置为 `true` 时，事件实例将保留对订阅服务器实例的强引用，因此不允许它进行垃圾回收。有关如何取消订阅的信息，请参阅本主题后面的取消订阅事件部分。
- 当设置为 `false` （省略此参数时的默认值） 时，该事件将保持对订阅服务器实例的弱引用，从而允许垃圾回收器在没有其他引用时释放订阅服务器实例。收集订阅服务器实例后，将自动取消订阅该事件。

## 取消订阅 Event

如果订阅者不想再接收事件，可以使用订阅者的处理程序取消订阅，也可以使用订阅令牌取消订阅。

下面的代码示例演示如何直接取消订阅处理程序。

```cs
public class MainPageViewModel
{
    TickerSymbolSelectedEvent _event;
    public MainPageViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent<TickerSymbolSelectedEvent>();
        _event.Subscribe(ShowNews);
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(ShowNews);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```

下面的代码示例演示如何使用订阅令牌取消订阅。令牌作为 `Subscribe` 方法的返回值提供。

```cs
public class MainPageViewModel
{
    TickerSymbolSelectedEvent _event;
    SubscriptionToken _token;
    public MainPageViewModel(IEventAggregator ea)
    {
        _event = ea.GetEvent<TickerSymbolSelectedEvent>();
        _token = _event.Subscribe(ShowNews);
    }

    void Unsubscribe()
    {
        _event.Unsubscribe(_token);
    }

    void ShowNews(string companySymbol)
    {
        //implement logic
    }
}
```
