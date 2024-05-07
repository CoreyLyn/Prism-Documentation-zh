# 将事件绑定到命令

该 ```InvokeCommandAction``` 类提供了一种方便的方法，用于在 XAML 中根据避免代码隐藏的 MVVM 范例将事件“绑定”到 ```ICommand``` 属性。

## 属性

```InvokeCommandAction``` 公开以下属性：

* ```Command``` 标识调用时要执行的命令, 这是必需的。
* ```AutoEnable``` 根据命令的 ```CanExecute``` ，这是一个可选字段，默认值为 ```True``` 。
* ```CommandParameter``` 标识要提供给命令的命令参数，这是一个可选字段。
* ```TriggerParameterPath``` 标识要分析的事件提供的对象中的路径，以标识要用作命令参数的子属性。

## 用法

### 基本用法

首先，需要通过指定 ```InteractionTrigger``` 。这是 WPF 中标准的现成功能。添加命名空间以便能够在 XAML 中声明它。

`xmlns:i="http://schemas.microsoft.com/xaml/behaviors"`

添加  `Prism` 命名空间以便能够在 XAML 中声明 `InvokeCommandAction` 。

`xmlns:prism="http://prismlibrary.com"`

并使用所需事件附加到控件。

```xml
<Window x:Class="UsingInvokeCommandAction.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
        xmlns:prism="http://prismlibrary.com/"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <Grid>
        <ListBox ItemsSource="{Binding Items}" SelectionMode="Single">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="SelectionChanged">
                    <prism:InvokeCommandAction Command="{Binding SelectedCommand}"
                                               CommandParameter="{Binding MyParameter}" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </ListBox>
    </Grid>
</Window>
```

### TriggerParameterPath

在下面的代码中， ```SelectionChanged``` 事件接收到一个包含名为 ```AddedItems``` 的 ```IList``` 属性的 ```SelectionChangedEventArgs``` 对象。使用 ```TriggerParameterPath``` 指定此属性，以便将其作为参数传递给 ```ICommand``` 对象。

```xml
<Window x:Class="UsingInvokeCommandAction.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
        xmlns:prism="http://prismlibrary.com/"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <Grid>
        <ListBox ItemsSource="{Binding Items}" SelectionMode="Single">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="SelectionChanged">
                    <prism:InvokeCommandAction Command="{Binding SelectedCommand}"
                                               CommandParameter="{Binding MyParameter}"
                                               TriggerParameterPath="AddedItems" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </ListBox>
    </Grid>
</Window>
```

### AutoEnable

```AutoEnable``` 属性指定是否应根据 ```ICommand.CanExecute``` 的结果自动启用或禁用关联元素。默认值为 ```true``` ，因为这是最常见的用法。

```xml
<Window x:Class="UsingInvokeCommandAction.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
        xmlns:prism="http://prismlibrary.com/"
        prism:ViewModelLocator.AutoWireViewModel="True"
        Title="{Binding Title}" Height="350" Width="525">
    <Grid>
        <ListBox ItemsSource="{Binding Items}" SelectionMode="Single">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="SelectionChanged">
                    <prism:InvokeCommandAction Command="{Binding SelectedCommand}"
                                               CommandParameter="{Binding MyParameter}"
                                               TriggerParameterPath="AddedItems"
                                               AutoEnable="true" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </ListBox>
    </Grid>
</Window>
```

## 完整代码示例

有关完整的代码示例，请转到 [GitHub](https://github.com/PrismLibrary/Prism-Samples-Wpf) 中的 ***Prism-Samples-Wpf*** 存储库并参考 [29-InvokeCommandAction](https://github.com/PrismLibrary/Prism-Samples-Wpf/tree/master/29-InvokeCommandAction).
