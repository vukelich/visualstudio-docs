---
title: "Open Folder Extensibility Overview | Microsoft Docs"
ms.custom: ""
ms.date: "02/21/2018"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "vs-ide-sdk"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 154825cf-901a-4957-ad8b-bad5403c30c2
caps.latest.revision: 1
author: "svukel"
ms.author: "svukel"
manager: "viveis"
ms.workload: 
  - "vssdk"
---

# Open Folder Extensibility

## What is Open Folder?

Open Folder is an experience introduced in Visual Studio 2017 allowing users to open any code base. Without any workloads installed, Open Folder provides Solution Explorer tree population and search, editor colorization, GoTo and Find in Files search, and more. Other workloads, like the workloads for .NET and C++ development, power richer Intellisense and specific language-specific functionality.

### A divergence from project systems
Historically, Visual Studio only understood the collection of code in a Solution and its projects. A project system is responsible for the functionality and user interactions of a loaded project. The project system understands what files its project contains, the visual representation of the project contents, dependencies on other projects, and modification of the underlying project file. It's through these hierarchies and capabilities that other components can do some work on behalf of the user. Not all user code bases are represented in this structure. Scripting language projects where there is nothing 'buildable' are an example. With Open Folder, Visual Studio gives end users a new way of interacting with any of their code.

New APIs under the `Microsoft.VisualStudio.Workspace.*` are available for extenders to produce and consume data or actions around files within Open Folder. New concepts or different extension point areas include:
- File contexts and actions
- Workspace indexing
- Tasks.vs.json and Launch.vs.json
- Workspace settings

### Workspaces
A workspace is how Visual Studio represents an arbitrary collection of files in Open Folder. By itself, the Open Folder feature doesn't understand the contents or features related to files within the folder. Rather, it provides a general set of APIs for features and extensions to produce and consume data that others can act upon. The producers are composed through MEF using various export attributes.

>!NOTE
>The term "workspace" is as both the term for some set of hierarchical set of files and folders as well as the informal term for an instance of the `IWorkspace` type. The Open Folder feature in Visual Studio is one application of this concept.

#### Workspace providers and services
Workspace providers and services provide the data and functionality to react to the contents of a workspace. They might provide contextual file information, symbols in source files, or build functionality.

One difference between providers and services is their relation to the workspace. A workspace can have many providers of a particular type, but only one service of a particular type is created per workspace. For example, there are many file scanner providers because each map to a different language or file extension they parse while there is only one indexing service per workspace.

Another key difference is consumption of data from providers and services. Extensions can get instances of and interact with workspaces services. Extension methods on `IWorkspace` are available for the services provided by Visual Studio, such as [IWorkspace.GetFileWatcherService][Doc:WorkspaceServiceHelper.GetFileWatcherService]. Do not author services that conflict with Visual Studio as that will lead to unexpected issues. Your extension may offer a workspace service for components within your extension or for other extensions to consume. Consumers should use [IWorkspace.GetServiceAsync][Doc:WorkspaceServiceHelper.GetServiceAsync] or an extension method you provide on the `IWorkspace` type.

Both concepts use a factory pattern and are imported from MEF by the workspace. All export attributes implement `IProviderMetadataBase` or `IWorkspaceServiceFactoryMetadata`, but there are sub-types specific for the type being exported. 


#### Related types
- [IWorkspace][Doc:IWorkspace] is the central entity for an opened workspace like an opened folder.
- [IWorkspaceProviderFactory\<T\>][Doc:IWorkspaceProviderFactory<T>] creates a provider per workspace instantiated.
- [IWorkspaceServiceFactory\<T\>][Doc:IWorkspaceServiceFactory<T>] creates a service per workspace instantiated.
- [WorkspaceServiceHelper][Doc:WorkspaceServiceHelper] provides helper methods for accessing well-known services or arbitrary services.

### File contexts
All insights in workspaces are produced by "file context providers" that implement the `IFileContextProvider` interface. These extensions might look for patterns in folders or files, read MSBuild files, makefiles, package dependencies, etc. in order to accumulate the insights they need to define a file context. A file context by itself does not perform any action, but rather provides data that another kind of extension can then act on.

Each `FileContext` has a `Guid` associated with it that identifies the type of data it carries. AnyCode uses this `Guid` later to match it up with extensions that may consume the data. A file context provider is exported with metadata that identifies which file context `Guid`s it may produce data for.

Once contrived, a file context can be associated with any number of files or folders in the workspace. A given file or folder may be associated with any number of file contexts. It is a many-to-many relationship.

The most common scenarios for file contexts are related to build, debug, and language services. For more information, see other sections related to these scenarios.

#### `FileContext` lifecycle
Lifecycles for a `FileContext` are non-deterministic. At any time, a component can asynchronously request for some set of context types, and the appropriate producers respond with any of the context types they support. The `IWorkspace` instance acts as the middle-man between the consumer and producers through the `IWorkspace.GetFileContextsAsync` method. Consumers might request a context and perform  However, changes might happen to files that cause a file context to become outdated. For example, if a build context is provided for some file but an on-disk change invalidates that context, then the original producer can invoke `FileContext.OnFileContextChanged`. Any consumers still referencing that `FileContext` can then requery for a new `FileContext`.

>[!NOTE]
> There is no push model to consumers. Consumers won't be notified of a provider's new `FileContext` if one becomes available.

#### Expensive file context computations
Reading contents from the disk can be expensive, especially when you're trying to find the relation between files. Say you're queried for a file context for a source file, but the file context is dependent on metadata from a project file. Parsing the project file or evaluating it with MSBuild is expensive. Instead, you could provide an `IFileScanner` to create `FileDataValue` data during workspace indexing. For more information on indexing, see the [Workspace indexing](#Workspace-indexing) section.

>![Warning]
>Be cautious of other ways your `FileContext` might be expensive to compute. Some UI interactions are synchronous and rely on a high volume of `FileContext` requests. A good examples include opening an editor tab and opening a **Solution Explorer** context menu. These actions make many build context type requests.

#### Related types and methods
- [FileContext][Doc:FileContext] holds data and metadata.
- [IFileContextProvider][Doc:IFileContextProvider] and [IFileContextProvider\<T\>][Doc:IFileContextProvider<T>] create the file context.
- [ExportFileContextProviderAttribute][Doc:ExportFileContextProviderAttribute] exports a file context provider.
- [IWorkspace.GetFileContextsAsync][Doc:IWorkspace.GetFileContextsAsync] is used for consumers to get file contexts.
- [BuildContextTypes][Doc:BuildContextTypes] defines build context types that Open Folder will consume.

### Workspace indexing
In a Solution, project systems are responsible for providing funcitonality for build, debug, **GoTo** symbol searching, and more. Project sysetms can do this work because they understand the relation and capabilities of files within a project. A workspace needs the same insight to provide rich IDE features, too. The collection and persistent storage of this data is a process called workspace indexing. This indexed data can be queried through a set of asynchronous APIs. Extenders can participate in the indexing process by providing `IFileScanner`s that know how to handle certain types of files.

#### Types of indexed data
There are several types of indexed data. 

`FileSystemEntity` is the basic 



#### Querying for indexed data
There are two asynchronous types available to access persisted data. The first is through `IIndexWorkspaceData`. It provides basic access to a single file's `FileReferenceResult` and `FileDataResult` data.

```charp
using Microsoft.VisualStudio.Workspace;
using Microsoft.VisualStudio.Workspace.Indexing;

private static IIndexWorkspaceData GetCachedIndexedData(IWorkspace workspace)
{
    // Gets access to indexed data wrapped in a cache.
    return workspace?.GetIndexWorkspaceDataService()?.CreateIndexWorkspaceData();
}

private static IIndexWorkspaceService GetDirectIndexedData(IWorkspace workspace)
{
    // Gets direct access to indexed data.
    // Can also be casted to IIndexWorkspaceService2.
    return workspace?.GetIndexWorkspaceService();
}
```

#### Particpating in indexing
Workspace indexing follows the following sequence:
1) Discovery and persistence of file system entities in the workspace (only on initial opening scan)
1) 

### Tasks.vs.json and Launch.vs.json

>![TIP]
>For more information on authoring a tasks.vs.json or launch.vs.json file, see [Customize build and debug tasks][Ref:TasksLaunchSettings].

### Workspace settings

### Detailed scenarios

#### File watching
A workspace listens to file change notifications and provides the [IFileWatcherService][Doc:IFileWatcherService] via [IWorkspace.GetFileWatcherService][Doc:WorkspaceServiceHelper.GetFileWatcherService]. Events are filtered to items important to a workspace and 

#### Build support
Up to date/incremental

Progress

#### Debug support
Set as default

File contexts

File data values

#### Language service support
A language service might self-activate based on the file extension or content of an opened document, this "loose file" language service is limited to syntax highlighting. Additional info is required to provide a richer experience when editing/reviewing code. Each language service has its own API for initialization with this extra contextual data for a document. This is typically managed by a project system, which is tightly coupled both to the language service and to the build system.

In Open Folder, language services are initialized by an `WorkspaceSymbolSearchAsync` extension point that specializes only in that language service and knows nothing of the build authoring. In this way, a language service owner can maintain a single Open Folder extension regardless of how many patterns exist within folders and files for running their compiler during a build (e.g. MSBuild, makefiles, etc.). When files from which a file context was created are changed on disk and the file context is refreshed, the language service provider is notified of the updated set of file contexts. The language service provider can then update its model.

When a document is opened in the editor, Visual Studio only considers language service providers that require file context types for which a matching file context provider can be found. It then passes the file context(s) from the matching provider(s) to the selected language service provider via `ILangaugeServiceProvider.InitializeAsync`. What the language service provider does with that file context data is an implementation detail of the language service provider, but the expected user experience is a richer language service for that opened document.

##### How to
TO DO make example instead of this description

Define a Context class that describe the data asscociated to a file that will help a language service to consume it
Create a file context provider factory to produce File context that can be connected to your language service provider

Create a language service provider factory that is prepared to receive your context data of a file context and so something useful with it

##### Related interfaces
- [ILanguageServiceProvider][Doc:ILanguageServiceProvider]

### Do's and Don't's in extension authoring

- **DO** Return objects from `IWorkspaceProviderFactory.CreateProvider` or similar APIs that remember their `Workspace` context when created. Providers interfaces are written expecting this object is kept on creation. 
    - One scenario that might have an exception is `ILanguageServiceProvider`. Its methods are passed full paths instead of the typical workspace-relative paths.


### Feedback, comments, issues
Open Folder and the `Microsoft.VisualStudio.Workspace.*` APIs are under active development. If you see unexpected behavior, then see the known issues for the release of interest. The Developer Community is the recommended place to vote and create any issues. For each feedback, we highly recommend a detailed description of the Visual Studio version you're developing for, the APIs you're using (both what you've implemented and services you're interacting with), the expected outcome, and the actual outcome. Use GitHub's issue tracking for giving feedback on this and related documentation.

[Doc:BuildContextTypes]:/dotnet/api/microsoft.visualstudio.workspace.build.buildcontexttypes
[Doc:ExportFileContextProviderAttribute]:/dotnet/api/microsoft.visualstudio.workspace.ExportFileContextProviderAttribute
[Doc:FileContext]:/dotnet/api/microsoft.visualstudio.workspace.filecontext
[Doc:IFileContextProvider]:/dotnet/api/microsoft.visualstudio.workspace.ifilecontextprovider
[Doc:IFileContextProvider<T>]:/dotnet/api/microsoft.visualstudio.workspace.ifilecontextprovider-1
[Doc:IFileWatcherService]:/dotnet/api/microsoft.visualstudio.workspace.ifilewatcherservice
[Doc:IIndexWorkspaceData]:/dotnet/api/microsoft.visualstudio.workspace.indexing.iindexworkspacedata
[Doc:IIndexWorkspaceDataService.CreateIndexWorkspaceData]:/dotnet/api/microsoft.visualstudio.workspace.indexing.iindexworkspacedataservice.createindexworkspacedata
[Doc:ILanguageServiceProvider]:/dotnet/api/microsoft.visualstudio.workspace.intellisense.ilanguageserviceprovider
[Doc:IWorkspace]:/dotnet/api/microsoft.visualstudio.workspace.iworkspace
[Doc:IWorkspace.GetFileContextsAsync]:/dotnet/api/microsoft.visualstudio.workspace.iworkspace.getfilecontextsasync
[Doc:IWorkspaceProviderFactory<T>]:/dotnet/api/microsoft.visualstudio.workspace.iworkspaceproviderfactory-1
[Doc:IWorkspaceServiceFactory<T>]:/dotnet/api/microsoft.visualstudio.workspace.iworkspaceservicefactory-1
[Doc:WorkspaceServiceHelper]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper
[Doc:WorkspaceServiceHelper.GetFileWatcherService]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper.getfilewatcherservic
[Doc:WorkspaceServiceHelper.GetServiceAsync]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper.getserviceasync
[Ref:TasksLaunchSettings]:/ide/customize-build-and-debug-tasks-in-visual-studio