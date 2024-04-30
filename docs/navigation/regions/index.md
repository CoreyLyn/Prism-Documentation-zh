# Getting Started

?> 作为 Prism 9.0 计划的一部分，我们非常重视在所有受支持的平台上统一 Prism API。因此，区域抽象不再是特定于平台的。这大大简化了从一个平台过渡到另一个平台时必须学习的内容。作为额外的好处，这意味着你现在可以生成跨 WPF、.NET MAUI 和 Uno 平台共享 ViewModel 的应用程序，甚至可以将 Xamarin.Forms 应用程序代码移植到这些其他平台。

当用户与富客户端应用程序交互时，其用户界面 （UI） 将不断更新，以反映用户正在处理的当前任务和数据。随着用户与应用程序交互并完成应用程序中的各种任务，UI 可能会随着时间的推移而发生相当大的变化。应用程序协调这些 UI 更改的过程通常称为导航。本主题介绍如何使用 Prism 库实现复合模型-视图-视图模型 （MVVM） 应用程序的导航。

通常，导航意味着删除 UI 中的某些控件，同时添加其他控件。在其他情况下，导航可能意味着更新一个或多个现有控件的可视状态。例如，当应用状态发生变化时，某些控件可能只是隐藏或折叠，而其他控件则显示或展开。导航还可能意味着控件显示的数据将更新以反映应用程序的当前状态。例如，在大从-从场景下，将根据大图中当前选定的项目更新明细视图中显示的数据。所有这些方案都可以被视为导航，因为用户界面已更新以反映用户的当前任务和应用程序的当前状态。

应用程序中的导航可以由用户与 UI 的交互（通过鼠标事件或其他 UI 手势）产生，也可以由应用程序本身作为内部逻辑驱动的状态更改的结果。在某些情况下，导航可能涉及非常简单的 UI 更新，不需要自定义应用程序逻辑。在其他情况下，应用程序可以实现复杂的逻辑，以编程方式控制导航，以确保强制执行某些业务规则，例如，应用程序可能不允许用户在未首先确保输入的数据正确的情况下离开特定窗体。

在 Windows Presentation Foundation （WPF） 或 UNO 应用程序中实现所需的导航行为通常相对简单，因为它提供对导航的直接支持。但是，在使用模型-视图-视图模型 （MVVM） 模式的应用程序或使用多个松散耦合模块的复合应用程序中实现导航可能更复杂。Prism 提供了在这些情况下实现导航的指导。

## Prism 中的导航

导航被定义为应用程序协调由于用户与应用程序的交互或内部应用程序状态更改而对其 UI 的更改的过程。

| 导航类型 | 描述 |
|-----------------|-------------|
| State Based     | Navigation accomplished via state changes to existing controls in the visual tree. |
| View Based      | Navigation accomplished via the addition or removal of elements from the visual tree. |

Prism 提供了有关实现这两种导航样式的指导，重点介绍应用程序使用 Model-View-ViewModel （MVVM） 模式将 UI（封装在视图中）与表示逻辑和数据（封装在视图模型中）分开的情况。

| 主题                            | 描述 |
|-----------------------------------|-------------|
| [Basic Region Navigation](xref:Navigation.Regions.BasicRegionNavigation) | Get started with the Prism navigation system. |
| [Confirming Navigation](xref:Navigation.Regions.ConfirmingNavigation) | Learn how to allow the user to interact with the navigation system. |
| [Controlling View Lifetime](xref:Navigation.Regions.ControllingViewLifetime) | Setup your view to remain in memory even after navigated away from. |
| [Navigate Existing Views](xref:Navigation.Regions.NavigationExistingViews) | Navigate between active views. |
| [Navigation Journal](xref:Navigation.Regions.NavigationJournal) | Use the navigation journal to allow the user to navigate from within the view. |
| [Passing Parameters](xref:Navigation.Regions.PassingParameters) | Pass data to the view being navigated to. |
| [View and View Model Participation](xref:Navigation.Regions.ViewViewModelParticipation) | Link your views and view models to the navigation system. |
