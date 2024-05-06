# 使用 Prism 库进行模块化应用程序开发

模块化应用程序是划分为一组松散耦合的功能单元（命名模块）的应用程序，这些功能单元可以集成到更大的应用程序中。客户端模块封装了应用程序整体功能的一部分，通常表示一组相关关注点。它可以包括相关组件的集合，例如应用程序功能（包括用户界面和业务逻辑），也可以包括应用程序基础结构（例如用于记录或验证用户身份的应用程序级服务）。模块彼此独立，但可以以松耦合的方式相互通信。使用模块化应用程序设计可以更轻松地开发、测试、部署和维护应用程序。

例如，考虑个人银行应用程序。用户可以访问各种功能，例如在账户之间转账、支付账单以及从单个用户界面 （UI） 更新个人信息。然而，在幕后，这些功能中的每一个都封装在一个离散的模块中。这些模块相互通信，并与后端系统（如数据库服务器和 Web 服务）通信。应用程序服务将各种组件集成到每个不同的模块中，并处理与用户的通信。用户看到的是一个看起来像单个应用程序的集成视图。

下图显示了具有多个模块的模块化应用程序的设计。

![Module composition](../images/ModularityAppArchitecture.png)

## 构建模块化应用程序的好处

您可能已经在使用程序集、接口和类构建架构良好的应用程序，并采用良好的面向对象设计原则。即便如此，除非非常小心，否则您的应用程序设计可能仍然是“整体式”的（所有功能在应用程序中以紧密耦合的方式实现），这可能会使应用程序难以开发、测试、扩展和维护。

另一方面，模块化应用程序方法可以帮助您识别应用程序的大规模功能区域，并允许您独立开发和测试该功能。这可以使开发和测试更容易，但也可以使您的应用程序更灵活，更容易在将来扩展。模块化方法的好处是，它可以使整个应用程序体系结构更加灵活和可维护，因为它允许您将应用程序分解为可管理的部分。每个部分都封装了特定的功能，每个部分都通过清晰但松散耦合的通信渠道进行集成。

## Prism 对模块化应用开发的支持

Prism 为模块化应用程序开发和应用程序中的运行时模块管理提供支持。使用 Prism 的模块化开发功能可以节省您的时间，因为您不必实施和测试自己的模块化框架。Prism 支持以下模块化应用程序开发功能：

- 用于注册命名模块和每个模块位置的模块目录;您可以通过以下方式创建模块目录：
  - 通过在代码或可扩展应用程序标记语言 （XAML） 中定义模块
  - 通过发现目录中的模块，这样您就可以加载所有模块，而无需在集中式目录中显式定义
  - 通过在配置文件中定义模块
  - 模块的声明性元数据属性，以支持初始化模式和依赖项
- 对于模块加载：
  - 依赖关系管理，包括重复和周期检测，以确保模块以正确的顺序加载，并且只加载和初始化一次
  - 模块的按需和后台下载，以最大限度地减少应用程序启动时间;其余模块可以在后台或需要时加载和初始化
- 与依赖注入容器集成，支持模块之间的松耦合

## 核心概念

本节介绍 Prism 中与模块化相关的核心概念，包括 ```IModule``` 接口、模块加载过程、模块目录、模块之间的通信以及依赖注入容器。

### IModule：模块化应用程序的构建块

模块是功能和资源的逻辑集合，其打包方式可以单独开发、测试、部署和集成到应用程序中。包可以是一个或多个程序集。每个模块都有一个中心类，该类负责初始化模块并将其功能集成到应用程序中。该类实现 ```IModule``` 接口.

_**注意:** 一个实现了  ```IModule``` 接口的类的存在就足以将该包识别为一个模块。_

The ```IModule``` interface has two methods, named ```OnInitialized``` and ```RegisterTypes```. Both take a reference to the dependency injection container as a parameter. When a module is loaded into the application, ```RegisterTypes``` is called first and should be used to register any services or functionality that the module implements. Next the ```OnInitialized``` method is called. It is here that things like view registrations or any other module initialization code should be performed.

```cs
public class MyModule : IModule
{
    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
        // register with the container that SomeService implements ISomeService
        // ISomeService is defined in the Infrastructure module, see app architecture diagram
        containerRegistry.Register<MyApplication.Infrastructure.ISomeService, SomeService>();
    }

    public void OnInitialized(IContainerProvider containerProvider)
    {
        // use the containerProvider to retrieve the instance of the Prism RegionManager
        // and register the view in this module with a specific region in the app
        var regionManager = containerProvider.Resolve<IRegionManager>();
        regionManager.RegisterViewWithRegion("MyModuleView", typeof(Views.ThisModuleView));
    }
}
```

### Module Lifecycle

The module loading process in Prism includes the following sequence:

- **Registering** Modules are created by implementing the IModule interface inside of a class.
- **Discovering modules**. The modules to be loaded at run-time for a particular application are defined in a Module catalog. The catalog contains information about the modules to be loaded, such as their location, and the order in which they are to be loaded.
- **Loading modules**. The assemblies that contain the modules are loaded into memory.
- **Initializing modules**. The modules are then initialized. This means creating instances of the module class and calling the ```RegisterTypes``` and ```OnInitialized``` methods on them via the ```IModule``` interface.


### Module Catalog

The ```ModuleCatalog``` holds information about the modules that can be used by the application. The catalog is essentially a collection of ```ModuleInfo``` classes. Each module is described in a ```ModuleInfo``` class that records the name, type, and location, among other attributes of the module. There are several typical approaches to filling the **ModuleCatalog** with **ModuleInfo** instances:

- Registering modules in code
- Registering modules in XAML
- Registering modules in a configuration file
- Discovering modules in a local directory on disk

The registration and discovery mechanism you should use depends on what your application needs. Using a configuration file or XAML file allows your application to not require references to the modules. Using a directory can allow an application to discover modules without having to specify them in a file.

### Controlling When to Load a Module

Prism applications can initialize modules as soon as possible, known as "when available," or when the application needs them, known as "on-demand." Consider the following guidelines for loading modules:

- Modules required for the application to run must be loaded with the application and initialized when the application runs.
- Modules containing features that are rarely used (or are support modules that other modules optionally depend upon) can be loaded and initialized on-demand.

Consider how you are partitioning your application, common usage scenarios and application start-up time for determining how to configure your app for initialization.

### Integrate Modules With The Application

Each of the ```Prism.DryIoc.[Platform]``` and ```Prism.Unity.[Platform]``` assemblies provide an ```Application``` based class that is used as the base class for the App class. Override the virtual method ```CreateModuleCatalog``` to create the desired type of module catalog.

For each of the modules in the app, implement the ```IModuleInfo``` interface to register module types and services. The following are common things to do to when integrating a module into the app:

- Add the module's views to the application's navigation structure. This is common when building composite UI applications using view discovery or view injection.
- Subscribe to application level events or services.
- Register shared services with the application's dependency injection container.

### Communicate Between Modules

Even though modules should have low coupling between each other, it is common for modules to communicate with each other. There are several loosely coupled communication patterns, each with their own strengths. Typically, combinations of these patterns are used to create the resulting solution. The following are some of these patterns:

- **Loosely coupled events**. A module can broadcast that a certain event has occurred. Other modules can subscribe to those events so they will be notified when the event occurs. Loosely coupled events are a lightweight manner of setting up communication between two modules; therefore, they are easily implemented. However, a design that relies too heavily on events can become hard to maintain, especially if many events have to be orchestrated together to fulfill a single task. In that case, it might be better to consider a shared service.
- **Shared services**. A shared service is a class that can be accessed through a common interface. Typically, shared services are found in a shared assembly and provide system-wide services, such as authentication, logging, or configuration.
- **Shared resources**. If you do not want modules to directly communicate with each other, you can also have them communicate indirectly through a shared resource, such as a database or a set of web services.

### Dependency Injection and Modular Applications

Containers like **Unity** and **DryIoc** allow you to easily use Inversion of Control (IoC) and Dependency Injection, which are powerful design patterns that help to compose components in a loosely-coupled fashion. It allows components to obtain references to the other components that they depend on without having to hard code those references, thereby promoting better code re-use and improved flexibility. Dependency injection is very useful when building a loosely coupled, modular application. Prism is designed to be agnostic about the dependency injection container used to compose components within an application.

Regardless of which of the three containers is chosen, Prism will use the container to construct and initialize each of the modules so that they remain loosely coupled.

## Key Decisions

The first decision you will make is whether you want to develop a modular solution. There are numerous benefits of building modular applications as discussed in the previous section, but there is a commitment in terms of time and effort that you need to make to reap these benefits. If you decide to develop a modular solution, there are several more things to consider:

- **Determine the framework you will use**. You can create your own modularity framework, use Prism, or another framework.
- **Determine how to organize your solution**. Approach a modular architecture by defining the boundaries of each module, including what assemblies are part of each module. You can decide to use modularity to ease the development, as well as to have control over how the application will be deployed or if it will support a plug-in or extensible architecture.
- **Determine how to partition your modules**. Modules can be partitioned differently based on requirements, for example, by functional areas, provider modules, development teams and deployment requirements.
- **Determine the core services that the application will provide to all modules**. An example is that core services could be an error reporting service or an authentication and authorization service.
- **If you are using Prism, determine what approach you are using to register modules in the module catalog**. For WPF, you can register modules in code, XAML, in a configuration file, or discovering modules in a local directory on disk.
- **Determine your module communication and dependency strategy**. Modules will need to communicate with each other, and you will need to deal with dependencies between modules.
- **Determine your dependency injection container**. Typically, modular systems require dependency injection, inversion of control, or service locator to allow the loose coupling and dynamic loading and creating of modules. Prism allows a choice between using Unity or DryIoc and provides libraries for Unity and DryIoc based applications.
- **Minimize application startup time**. Think about on-demand and background downloading of modules to minimize application startup time.
- **Determine deployment requirements**. You will need to think about how you intend to deploy your application.

The next sections provide details about some of these decisions.

## Partition Your Application into Modules

When you develop your application in a modularized fashion, you structure the application into separate client modules that can be individually developed, tested, and deployed. Each module will encapsulate a portion of your application's overall functionality. One of the first design decisions you will have to make is to decide how to partition your application's functionality into discrete modules.

A module should encapsulate a set of related concerns and have a distinct set of responsibilities. A module can represent a vertical slice of the application or a horizontal service layer.

![A vertical sliced application](../images/ModularityVertical.png)

An application with modules organized around vertical slices

![A horizontal layered application](../images/ModularityHor.png)

An application with modules organized around horizontal layers

A larger application may have modules organized with vertical slices and horizontal layers. Some examples of modules include the following:

- A module that contains a specific application feature, such as a module that serves news and/or announcements
- A module that contains a specific sub-system or functionality for a set of related use cases, such as purchasing, invoicing, or general ledger
- A module that contains infrastructure services, such as logging, caching, and authorization services, or web services
- A module that contains services that invoke line-of-business (LOB) systems, such as Siebel CRM and SAP, in addition to other internal systems

A module should have a minimal set of dependencies on other modules. When a module has a dependency on another module, it should be loosely coupled by using interfaces defined in a shared library instead of concrete types, or by using the **EventAggregator** to communicate with other modules via **EventAggregator** event types.

The goal of modularity is to partition the application in such a way that it remains flexible, maintainable, and stable even as features and technologies are added and removed. The best way to accomplish this is to design your application so that modules are as independent as possible, have well defined interfaces, and are as isolated as possible.

### Determine Ratio of Projects to Modules

There are several ways to create and package modules. The recommended and most common way is to create a single assembly per module. This helps keep logical modules separate and promotes proper encapsulation. It also makes it easier to talk about the assembly as the module boundary as well as the packaging of how you deploy the module. However, nothing prevents a single assembly from containing multiple modules, and in some cases this may be preferred to minimize the number of projects in your solution. For a large application, it is not uncommon to have 10–50 modules. Separating each module into its own project adds a lot of complexity to the solution and can slow down Visual Studio performance. Sometimes it makes sense to break a module or set of modules into their own solution to manage this if you choose to stick to one module per assembly/Visual Studio project.

## Use Dependency Injection for Loose Coupling

A module may depend on components and services provided by the host application or by other modules. Prism supports the ability to register dependencies between modules so that they are loaded and initialized in the right order. Prism also supports the initialization of modules when they are loaded into the application. During module initialization, the module can retrieve references to the additional components and services it requires, and/or register any components and services that it contains in order to make them available to other modules.

A module should use an independent mechanism to get instances of external interfaces instead of directly instantiating a concrete type, for example by using a dependency injection container or factory service. Dependency injection containers such as Unity or DryIoc allow a type to automatically acquire instances of the interfaces and types it needs through dependency injection. Prism integrates with Unity and DryIoc to allow a module to easily use dependency injection.

The following diagram shows the typical sequence of operations when modules are loaded that need to acquire or register references to the components and services.

![Example of dependency injection](../images/ModularityDi.png)

In this example, the ```OrdersModule``` assembly defines an ```OrdersRepository``` class (along with other views and classes that implement order functionality). The ```CustomerModule``` assembly defines a ```CustomersViewModel``` class which depends on the ```OrdersRepository```, typically based on an interface exposed by the service. The application startup and bootstrapping process contains the following steps:

1. The ```App``` class that is derived from ```PrismApplication``` starts the module initialization process, and the module loader loads and initializes the ```OrdersModule```.
1. In the initialization of the ```OrdersModule```, it registers the ```OrdersRepository``` with the container.
1. The module loader then loads the ```CustomersModule```. The order of module loading can be specified by the dependencies in the module metadata.
1. The ```CustomersModule``` constructs an instance of the ```CustomerViewModel``` by resolving it through the container. The ```CustomerViewModel``` has a dependency on the ```OrdersRepository``` (typically based on its interface) and indicates it through constructor or property injection. The container injects that dependency in the construction of the view model based on the type registered by the ```OrdersModule```. The net result is an interface reference from the ```CustomerViewModel``` to the ```OrderRepository``` without tight coupling between those classes.

**Note:** The interface used to expose the ```OrderRepository``` (```IOrderRepository```) could reside in a separate "shared services" assembly or an "orders services" assembly that only contains the service interfaces and types required to expose those services. This way, there is no hard dependency between the ```CustomersModule``` and the ```OrdersModule```.

> Note that both modules have an implicit dependency on the dependency injection container. This dependency is injected during module construction in the module loader.

## Core Scenarios

This section describes the common scenarios you will encounter when working with modules in your application. These scenarios include defining a module, registering and discovering modules, loading modules, initializing modules, specifying module dependencies, loading modules on demand, downloading remote modules in the background, and detecting when a module has already been loaded. You can register and discover modules in code, in a XAML or application configuration file, or by scanning a local directory.

### Defining a Module

A module is a logical collection of functionality and resources that is packaged in a way that can be separately developed, tested, deployed, and integrated into an application. Each module has a central class that is responsible for initializing the module and integrating its functionality into the application. That class implements the ```IModule``` interface, as shown here.

```cs
public class MyModule : IModule
{
    public void RegisterTypes(IContainerRegistry containerRegistry)
    {
    }

    public void OnInitialized(IContainerProvider containerProvider)
    {
    }
}
```
Implement ```RegisterTypes``` to handle the registration with the dependency injection container all of the services that this module implements.

How ```OnInitialized``` method is implemented will depend on the requirements of your application. Here is where you can register your views and do any other module level initialize that may be required.

### Registering and Discovering Modules

The modules that an application can load are defined in a module catalog. The Prism Module Loader uses the module catalog to determine which modules are available to be loaded into the application, when to load them, and in which order they are to be loaded.

The module catalog is represented by a class that implements the ```IModuleCatalog``` interface. The module catalog class is created by the ```PrismApplication``` base class during application initialization. Prism provides different implementations of module catalog for you to choose from. You can also populate a module catalog from another data source by calling the ```AddModule``` method or by deriving from ```ModuleCatalog``` to create a module catalog with customized behavior.

By default, the ```App``` class, derived from ```PrismApplication```, creates a ```ModuleCatalog``` in the ```CreateModuleCatalog``` method. In WPF and UNO, you can override this method to use different types of ```ModuleCatalog```.

#### Registering Modules in Code

The most basic module catalog, and default, is provided by the ```ModuleCatalog``` class. You can use this module catalog to programmatically register modules by specifying the module class type. You can also programmatically specify the module name and initialization mode. To register the module directly with the ```ModuleCatalog``` class, call the ```AddModule``` method in your application's ```PrismApplication``` derived ```App``` class. Override ```ConfigureModuleCatalog``` to add your modules. An example is shown in the following code.

```cs
protected override void ConfigureModuleCatalog()
{
    Type moduleCType = typeof(ModuleC);
    ModuleCatalog.AddModule(new ModuleInfo()
    {
        ModuleName = moduleCType.Name,
        ModuleType = moduleCType.AssemblyQualifiedName,
    });
}
```

> **Note:** If your application has a direct reference to the module type, you can add it by type as shown above; otherwise you need to provide the fully qualified type name and the location of the assembly.

To specify dependencies in code, use Prism supplied declarative attributes.

```cs
[Module(ModuleName = "ModuleA")]
[ModuleDependency("ModuleD")]
public class ModuleA : IModule
{
    ...
}
```

To specify on-demand loading in code, add the ```InitializationMode``` property to your new instance of ModuleInfo. Using the code below:

```cs
Type moduleCType = typeof(ModuleC);
ModuleCatalog.AddModule(new ModuleInfo()
{
    ModuleName = moduleCType.Name,
    ModuleType = moduleCType.AssemblyQualifiedName,
    InitializationMode = InitializationMode.OnDemand,
});
```

#### Registering Modules Using a XAML File

You can define a module catalog declaratively by specifying it in a XAML file. The XAML file specifies what kind of module catalog class to create and which modules to add to it. Usually, the .xaml file is added as a resource to your shell project. The module catalog is created by the App with a call to the ```CreateFromXaml``` method. From a technical perspective, this approach is very similar to defining the ```ModuleCatalog``` in code because the XAML file simply defines a hierarchy of objects to be instantiated.

The following code example shows a XAML file specifying a module catalog.

```xml
<--! ModulesCatalog.xaml -->
<Modularity:ModuleCatalog xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:sys="clr-namespace:System;assembly=mscorlib"
    xmlns:Modularity="clr-namespace:Microsoft.Practices.Prism.Modularity;assembly=Microsoft.Practices.Prism">

    <Modularity:ModuleInfoGroup Ref="file://DirectoryModules/ModularityWithMef.Desktop.ModuleB.dll" InitializationMode="WhenAvailable">
        <Modularity:ModuleInfo ModuleName="ModuleB" ModuleType="ModularityWithMef.Desktop.ModuleB, ModularityWithMef.Desktop.ModuleB, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    </Modularity:ModuleInfoGroup>

    <Modularity:ModuleInfoGroup InitializationMode="OnDemand">
        <Modularity:ModuleInfo Ref="file://ModularityWithMef.Desktop.ModuleE.dll" ModuleName="ModuleE" ModuleType="ModularityWithMef.Desktop.ModuleE, ModularityWithMef.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
        <Modularity:ModuleInfo Ref="file://ModularityWithMef.Desktop.ModuleF.dll" ModuleName="ModuleF" ModuleType="ModularityWithMef.Desktop.ModuleF, ModularityWithMef.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
            <Modularity:ModuleInfo.DependsOn>
                <sys:String>ModuleE</sys:String>
            </Modularity:ModuleInfo.DependsOn>
        </Modularity:ModuleInfo>
    </Modularity:ModuleInfoGroup>

    <!-- Module info without a group -->
    <Modularity:ModuleInfo Ref="file://DirectoryModules/ModularityWithMef.Desktop.ModuleD.dll" ModuleName="ModuleD" ModuleType="ModularityWithMef.Desktop.ModuleD, ModularityWithMef.Desktop.ModuleD, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</Modularity:ModuleCatalog>
```

> **Note:** ```ModuleInfoGroups``` provide a convenient way to group modules that are in the same assembly, are initialized in the same way, or only have dependencies on modules in the same group. Dependencies between modules can be defined within modules in the same ```ModuleInfoGroup```; however, you cannot define dependencies between modules in different ```ModuleInfoGroups```. Putting modules inside module groups is optional. The properties that are set for a group will be applied to all its contained modules. Note that modules can also be registered without being inside a group._

Example on creating the catalog from a XAML file is below:

```cs
protected override IModuleCatalog CreateModuleCatalog()
{
    return ModuleCatalog.CreateFromXaml(new Uri("/MyProject;component/ModulesCatalog.xaml", UriKind.Relative));
}
```

To specify dependencies in XAML, follow the example below:

```xml
<-- ModulesCatalog.xaml -->
<Modularity:ModuleInfo Ref="file://ModuleE.dll" moduleName="ModuleE" moduleType="ModuleE.Module, ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
<Modularity:ModuleInfo Ref="file://ModuleF.dll" moduleName="ModuleF" moduleType="ModuleF.Module, ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
    <Modularity:ModuleInfo.DependsOn>
        <sys:String>ModuleE</sys:String>
    </Modularity:ModuleInfo.DependsOn>
</Modularity:ModuleInfo>
```

To specify on-demand loading of your module, add the ```startupLoaded``` attribute to the ```Modularity:ModuleInfo``` element.

```xml
<Modularity:ModuleInfo Ref="file://ModuleE.dll" moduleName="ModuleE" moduleType="ModuleE.Module, ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" startupLoaded="false" />
```

#### Registering Modules Using a Configuration File

In WPF, it is possible to specify the module information in the App.config file. The advantage of this approach is that this file is not compiled into the application. This makes it very easy to add or remove modules at run time without recompiling the application.

The following code example shows a configuration file specifying a module catalog.

```xml
<!-- ModularityWithUnity.Desktop\\app.config -->
<xml version="1.0" encoding="utf-8" ?>
<configuration>
    <configSections>
        <section name="modules" type="Prism.Modularity.ModulesConfigurationSection, Prism.Wpf"/>
    </configSections>

    <modules>
        <module assemblyFile="ModularityWithUnity.Desktop.ModuleE.dll" moduleType="ModularityWithUnity.Desktop.ModuleE, ModularityWithUnity.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="ModuleE" startupLoaded="false" />
        <module assemblyFile="ModularityWithUnity.Desktop.ModuleF.dll" moduleType="ModularityWithUnity.Desktop.ModuleF, ModularityWithUnity.Desktop.ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="ModuleF" startupLoaded="false">
            <dependencies>
                <dependency moduleName="ModuleE"/>
            </dependencies>
        </module>
    </modules>
</configuration>
```

> **Note:** Even if your assemblies are in the global assembly cache or in the same folder as the application, the ```assemblyFile``` attribute is required. The attribute is used to map the ```moduleType``` to the correct ```IModuleTypeLoader``` to use.

In your application's ```App``` class, you need to specify that the configuration file is the source for your ```ModuleCatalog```. To do this, override the ```CreateModuleCatalog``` method and return an instance of the ```ConfigurationModuleCatalog``` class.

```cs
protected override IModuleCatalog CreateModuleCatalog()
{
    return new ConfigurationModuleCatalog();
}
```

> **Note:** You can still add modules to a ```ConfigurationModuleCatalog``` in code. You can use this, for example, to make sure that the modules that your application absolutely needs to function are defined in the catalog.

To specify dependencies in the app.config file:

```xml
<-- app.config -->
<modules>
    <module assemblyFile="ModuleE.dll" moduleType="ModuleE.Module, ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="moduleE" />
    <module assemblyFile="ModuleF.dll" moduleType="ModuleF.Module, ModuleF, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="moduleF">
        <dependencies>
            <dependency moduleName="moduleE" />
        </dependencies>
    </module>
</modules>
```

To specify on-demand loading using a configuration file, set the ```startupLoaded``` attribute of the ```module``` element to ```false```.

```xml
<module assemblyFile="ModularityWithUnity.Desktop.ModuleE.dll" moduleType="ModularityWithUnity.Desktop.ModuleE, ModularityWithUnity.Desktop.ModuleE, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" moduleName="ModuleE" startupLoaded="false" />
```

#### Discovering Modules in a Directory

The Prism ```DirectoryModuleCatalog``` class allows you to specify a local directory as a module catalog in WPF. This module catalog will scan the specified folder and search for assemblies that define the modules for your application. To use this approach, you will need to use declarative attributes on your module classes to specify the module name and any dependencies that they have. The following code example shows a module catalog that is populated by discovering assemblies in a directory.

```cs
protected override IModuleCatalog CreateModuleCatalog()
{
    return new DirectoryModuleCatalog() {ModulePath = @".\\Modules"};
}
```

To specify dependencies, use the same method as if you were using code.

To handle loading on demand or at startup, update the ```Module``` attribute as follows:
```cs
[Module(ModuleName = "ModuleA", OnDemand = true)]
[ModuleDependency("ModuleD")]
public class ModuleA : IModule
{
    ...
}
```

## Other Modularity Items of Note

### Requesting On-Demand loading of Module

After a module is specified as on-demand, the application can ask the module to be loaded. The code that wants to initiate the loading needs to obtain a reference to the ```IModuleManager``` service registered with the container in the ```App``` class.

An explicit load of a module can be performed by the following code:

```cs
public class SomeViewModel : BindableBase
{
    private IModuleManager _moduleManager = null;

    public SomeViewModel(IModuleManager moduleManager)
    {
        // use dependency injection to get the module manager
        _moduleManager = moduleManager;
    }

    private void LoadSomeModule(string moduleName)
    {
        _moduleManager.LoadModule(moduleName);
    }
}
```

### Detecting When a Module is Loaded

The ```ModuleManager``` service provides an event for applications to track when a module loads or fails to load.

```cs
public class SomeViewModel : BindableBase
{
    private IModuleManager _moduleManager = null;

    public SomeViewModel(IModuleManager moduleManager)
    {
        _moduleManager = moduleManager;
        _moduleManager.LoadModuleCompleted += _moduleManager_LoadModuleCompleted;
    }

    private void _moduleManager_LoadModuleCompleted(object sender, LoadModuleCompletedEventArgs e)
    {
        // ...
    }
}
```

To keep the application and modules loosely coupled, the application should avoid using this event to integrate the module with the application. Instead, the module's ```RegisterTypes``` and ```OnInitialized``` should handle integrating with the application.

The ```LoadModuleCompletedEventArgs``` contains an ```IsErrorHandled``` property. If a module fails to load and the application wants to prevent the ```ModuleManager``` from logging the error and throwing an exception, it can set this property to **true**.

> **Note**: After a module is loaded and initialized, the module assembly cannot be unloaded. The module instance reference will not be held by the Prism libraries, so the module class instance may be garbage collected after initialization is complete.
