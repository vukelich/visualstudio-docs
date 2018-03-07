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

## Introduction of Open Folder

Open Folder is a new experience introduced in Visual Studio 2017. It allows users to open any codebase. Without any workloads installed, Open Folder provides Solution Explorer tree population and search, editor colorization, GoTo and Find in Files search, and more. Other workloads, like the workloads for .NET and C++ development, power richer Intellisense and specific language-specific functionality.

### A divergence from project systems

Historically, Visual Studio only understood files in a Solution and its projects using project systems. A project system is responsible for the functionality and user interactions of a loaded project. It understands what files its project contains, the visual representation of the project contents, dependencies on other projects, and modification of the underlying project file. It's through these hierarchies and capabilities that other components can do some work on behalf of the user. Not all user codebases are represented in this structure. Scripting language projects where there is nothing 'buildable' are an example. With Open Folder, Visual Studio gives end users a new way of interacting with any of their code.

New APIs under the `Microsoft.VisualStudio.Workspace.*` are available for extenders to produce and consume data or actions around files within Open Folder. New concepts or different extension point areas include:

- File contexts and actions
- Workspace indexing
- Tasks.vs.json and Launch.vs.json
- Workspace settings

The following are examples of types **not** used in Open Folder due to incompatibility or the lack of project systems:

- `IVsHierarchy`
- `IVsProject`
- `DTE`

## Workspaces

A workspace is how Visual Studio represents an arbitrary collection of files in Open Folder. By itself, the Open Folder feature doesn't understand the contents or features related to files within the folder. Rather, it provides a general set of APIs for features and extensions to produce and consume data that others can act upon. The producers are composed through MEF using various export attributes.

>!NOTE
>The term "workspace" is as both the term for some set of hierarchical set of files and folders as well as the informal term for an instance of the `IWorkspace` type. The Open Folder feature in Visual Studio is one application of this concept.

### Workspace providers and services

Workspace providers and services provide the data and functionality to react to the contents of a workspace. They might provide contextual file information, symbols in source files, or build functionality.

Both concepts use a factory pattern and are imported by MEF by the workspace. All export attributes implement `IProviderMetadataBase` or `IWorkspaceServiceFactoryMetadata`, but there are sub-types specific for the type being exported.

One difference between providers and services is their relation to the workspace. A workspace can have many providers of a particular type, but only one service of a particular type is created per workspace. For example, there are many file scanner providers because each map to a different language or file extension they parse while there is only one indexing service per workspace.

Another key difference is consumption of data from providers and services. The workspace is the entry point to get data from providers for a couple reasons. First, providers typically have some narrow set of data they create. That might be symbols for a C# source file or build file contexts for a _CMakeLists.txt_ file. The workspace will match a consumers request to the providers whose metadata align with the request. Second, some scenarios allow for many providers to contribute to a requeset while others scenarios use the provider with highest priority.

In contrast, extensions can get instances of and interact directly with workspaces services. Extension methods on `IWorkspace` are available for the services provided by Visual Studio, such as [IWorkspace.GetFileWatcherService][Doc:WorkspaceServiceHelper.GetFileWatcherService]. Your extension may offer a workspace service for components within your extension or for other extensions to consume. Consumers should use [IWorkspace.GetServiceAsync][Doc:WorkspaceServiceHelper.GetServiceAsync] or an extension method you provide on the `IWorkspace` type.

>![Warning]
> Do not author services that conflict with Visual Studio. It will lead to unexpected issues.

### Disposal on workspace closure

On closure of a workspace, extenders might need to dipose but call asynchronous code. This can be done easily by writing providers and services that implement [IAsyncDisposable][Doc:IAsyncDisposable].

### Related types

- [IWorkspace][Doc:IWorkspace] is the central entity for an opened workspace like an opened folder.
- [IWorkspaceProviderFactory\<T\>][Doc:IWorkspaceProviderFactory<T>] creates a provider per workspace instantiated.
- [IWorkspaceServiceFactory\<T\>][Doc:IWorkspaceServiceFactory<T>] creates a service per workspace instantiated.
- [IAsyncDisposable][Doc:IAsyncDisposable] should be implemented on proviers and servies that need to run asynchronous code during disposal.
- [WorkspaceServiceHelper][Doc:WorkspaceServiceHelper] provides helper methods for accessing well-known services or arbitrary services.

## File contexts

All insights in workspaces are produced by "file context providers" that implement the `IFileContextProvider` interface. These extensions might look for patterns in folders or files, read MSBuild files, makefiles, package dependencies, etc. in order to accumulate the insights they need to define a file context. A file context by itself does not perform any action, but rather provides data that another extension can then act on.

Each `FileContext` has a `Guid` associated with it that identifies the type of data it carries. AnyCode uses this `Guid` later to match it up with extensions that may consume the data through the `FileContext.Context` property. A file context provider is exported with metadata that identifies which file context `Guid`s it may produce data for.

Once contrived, a file context can be associated with any number of files or folders in the workspace. A given file or folder may be associated with any number of file contexts. It's a many-to-many relationship.

The most common scenarios for file contexts are related to build, debug, and language services. For more information, see other sections related to these scenarios.

### `FileContext` lifecycle

Lifecycles for a `FileContext` are non-deterministic. At any time, a component can asynchronously request for some set of context types, and the appropriate producers respond with any of the context types they support. The `IWorkspace` instance acts as the middle-man between the consumer and producers through the `IWorkspace.GetFileContextsAsync` method. Consumers might request a context and perform some action based on the context. Others might request a context and maintain a long lived reference. However, changes might happen to files that cause a file context to become outdated. The producer of the can fire an event handler on the `FileContext` to notify consumers of updates. For example, if a build context is provided for some file but an on-disk change invalidates that context, then the original producer can invoke `FileContext.OnFileContextChanged`. Any consumers still referencing that `FileContext` can then requery for a new `FileContext`.

>[!NOTE]
> There is no push model to consumers. Consumers won't be notified of a provider's new `FileContext` if one becomes available.

### Expensive file context computations

Reading contents from the disk can be expensive, especially when a provider needs to find the relation between files. For example, a provider is queried for some source file's file context, but the file context is dependent on metadata from a project file. Parsing the project file or evaluating it with MSBuild is expensive. Instead, the extension can export an `IFileScanner` to create `FileDataValue` data during workspace indexing. Then when asked for file contexts, the `IFileContextProvider` can quickly query for that indexed data. the For more information on indexing, see the [Workspace indexing](#Workspace-indexing) section.

>![Warning]
>Be cautious of other ways your `FileContext` might be expensive to compute. Some UI interactions are synchronous and rely on a high volume of `FileContext` requests. Examples include opening an editor tab and opening a **Solution Explorer** context menu. These actions make many build context type requests.

### Related types and methods

- [FileContext][Doc:FileContext] holds data and metadata.
- [IFileContextProvider][Doc:IFileContextProvider] and [IFileContextProvider\<T\>][Doc:IFileContextProvider<T>] create the file context.
- [ExportFileContextProviderAttribute][Doc:ExportFileContextProviderAttribute] exports a file context provider.
- [IWorkspace.GetFileContextsAsync][Doc:IWorkspace.GetFileContextsAsync] is used for consumers to get file contexts.
- [BuildContextTypes][Doc:BuildContextTypes] defines build context types that Open Folder will consume.

## File context actions

While a `FileContext` itself is just data about some file(s), an `IFileContextAction` is a way to express and act on that data. `IFileContextAction` is flexible in its usage. Two of it's most common cases are building and debugging.

### Reporting progress

The [IFileContextAction.ExecuteAsync][Doc:IFileContextActionBase.ExecuteAsync] method is passed `IProgress<IFileContextActionProgressUpdate>`, but the argument shouldn't be used as that type. `IFileContextActionProgressUpdate` is an empty interface, and invoking `IProgress<IFileContextActionProgressUpdate>.Report(IFileContextActionProgressUpdate)` may throw `NotImplementedException`. Instead, the `IFileContextAction` must cast the argument to another type as necessary for the scenario.

For information on the types supplied by Visual Studio, see the respective scenario's documentation.

### `IFileContextAction` related APIs

- [IFileContextAction][Doc:IFileContextAction] executes in-proc or out of proc code based on a `FileContext`.
- [IFileContextActionProvider][Doc:IFileContextActionProvider] creates instances of `IFileContextAction`.
- [ExportFileContextActionProviderAttribute][Doc:ExportFileContextActionProviderAttribute] exports the type `IWorkspaceProviderFactory<IFileContextActionProvider>`.

## Workspace indexing

In a Solution, project systems are responsible for providing funcitonality for build, debug, **GoTo** symbol searching, and more. Project systems can do this work because they understand the relation and capabilities of files within a project. A workspace needs the same insight to provide rich IDE features, too. The collection and persistent storage of this data is a process called workspace indexing. This indexed data can be queried through a set of asynchronous APIs. Extenders can participate in the indexing process by providing `IFileScanner`s that know how to handle certain types of files.

### Types of indexed data

There are three kinds of data that are indexed. Note the type expected from file scanners differs from the type deserialized from the index.

|Data|File scanner type|Index query result type|Related types|
|--|--|--|--|
|References|`FileReferenceInfo`|`FileReferenceResult`|`FileReferenceInfoType`|
|Symbols|`SymbolDefinition`|`SymbolDefinitionSearchResult`|`ISymbolService` should be used instead of `IIndexWorkspaceService` for queries|
|Data values|`FileDataValue`|`FileDataResult<T>`||

### Querying for indexed data

There are two asynchronous types available to access persisted data. The first is through `IIndexWorkspaceData`. It provides basic access to a single file's `FileReferenceResult` and `FileDataResult` data.

```csharp
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

### Particpating in indexing

Workspace indexing rougly follows the following sequence:
1) Discovery and persistence of file system entities in the workspace (only on initial opening scan)
1) Per file, the matching provider with the highest priority is asked to scan for `FileReferneceInfo`s.
1) Per file, the matching provider with the highest priority is asked to scan for `SymbolDefinition`s.
1) Per file, all providers are aked for `FileDataValue`s.

Extensions can export a scanner by implementing `IWorkspaceProviderFactory<IFileScanner>` and exporting the type with `ExportFileScannerAttribute`. The `SupportedTypes` attribute argument should be one or more values from `FileScannerTypeConstants`. For an example scanner, see the VS SDK sample [here][Ex:FileScanner].

>![Warning]
>Do not export a file scanner that supports the `FileScannerTypeConstants.FileScannerContentType` type. It is used for Microsoft internal purposes, only.

In advanced situations, an extension might dynamically support an arbitrary set of file types. Rather than MEF exporting `IWorkspaceProviderFactory<IFileScanner>`, an extension can export `IWorkspaceProviderFactory<IFileScannerProvider>`. When indexing begins, this factory type will be imported, instantiated, and have its [IFileScannerProvider.GetSymbolScannersAsync][Doc:IFileScannerProvider.GetSymbolScannersAsync] method invoked. (`IFileScanner` instances supporting any value from `FileScannerTpeConstants` will be honored, not just symbols.)

## Tasks.vs.json and Launch.vs.json

>![TIP]
>For more information on authoring a tasks.vs.json or launch.vs.json file, see [Customize build and debug tasks][Ref:TasksLaunchSettings].

## Workspace settings

Workspaces have an `IWorkspaceSettingsManager` service with simple but powerful control over a workspace. For a basic overview of settings, see [Customize build and debug tasks][Ref:TasksLaunchSettings].

Settings for most `SettingsType` types are _.json_ files, such as _VSWorkspaceSettings.json_ and _tasks.vs.json_.

The power of workspace settings centers around "scopes," which are simply paths within the workspace. When a consumer calls `IWorkspaceSettingsManager.GetAggregatedSettings`, all the scopes that include the requested path and type of setting are aggregated. Scope aggregation priority is as follows:

1. "Local settings", which is typically the workspace root's `.vs` directory.
1. The requested path itself.
1. The parent directory of the requested path.
1. All further parent directories up to and including the workspace root.
1. "Global settings", which is in a user directory.

The result is an instance of `IWorkspaceSettings`. This object holds the settings for a particular type, and can be queried for setting key names stored as `string`. The `IWorkspaceSettings.GetProperty` methods and `IWorkspaceSettings` extension methods expect the caller to know the type of the setting value being requested. As most settings files are persisted as _.json_ files, many invocations will use `string`, `bool`, `int`, and arrays of those types. Object types are also supported. In those cases, you can use `IWorkspaceSettings` itself as the type argument. For example:

```json
{
  "intValue": 1,          // int
  "stringValue": "s",     // string
  "boolValue": true,      // bool
  "stringArray": [        // string[]
    "s1",
    "s2"
  ],
  "nested": {             // IWorkspaceSettings
    "nestedString": "ns"  // string from the 'nested' IWorkspaceSettings
  }
}
```

Assuming the above contents were in a user's _VSWorkspaceSettings.json_, the data can be accessed as:

```csharp
using System.Collections.Generic;
using Microsoft.VisualStudio.Workspace;
using Microsoft.VisualStudio.Workspace.Settings;

private static void ReadSettings(IWorkspace workspace)
{
    IWorkspaceSettingsManager settingsManager = workspace.GetSettingsManager();
    IWorkspaceSettings settings = settingsManager.GetAggregatedSettings(SettingsTypes.Generic);

    // result == WorkspaceSettingsResult.Success
    WorkspaceSettingsResult result = settings.GetProperty("intValue", out int intValue);
    result = settings.GetProperty("stringValue", out string stringValue);
    result = settings.GetProperty("boolValue", out bool boolValue);
    result = settings.GetProperty("stringArray", out string[] stringArray);
    result = settings.GetProperty("nested", out IWorkspaceSettings nested);
    result = nested.GetProperty("nestedString", out string nestedString);

    // Extension method alternative using default values.
    int intValueOrDefault = settings.Property("intValue", /* default */ 42);

    // Missing key. result == WorkspaceSettingsResult.Undefined
    result = settings.GetProperty("missing", out string missing);

    // Wrong type for a key. result == WorkspaceSettingsResult.Error
    result = settings.GetProperty("intValue", out IWorkspaceSettings notSettings);

    // Special ability to union "stringArray" across all scopes.
    IEnumerable<string> allStringArray = settings.UnionPropertyArray<string>("stringArray");
}
```

### Providing dynamic settings

Extensions can provide `IWorkspaceSettingsProvider`s. These in-memory providers allow extensions to add settings or override others.

Exporting an `IWorkspaceSettingsProvider` is different than other workspace providers. Many other providers will be imported when a condition is meet, like a file extension match.

```csharp
// The typical workspace provider factory pattern
[ExportMyProvider(some, args, to, export)]
internal class MyProviderFactory : IWorkspaceProviderFactory<IMyProvider>
{
     IMyProvider CreateProvider(IWorkspace workspace) => new MyProvider(workspace);
}

// IWorkspaceSettingsProvider pattern
[Export(typeof(IWorkspaceSettingsProviderFactory))]
internal class MySettingsProviderFactory : IWorkspaceSettingsProviderFactory
{
    // 100 is typically the value used by built-in settings providers. Lower value is higher priority.
    int Priority => 100;

    IWorkspaceSettingsProvider CreateSettingsProvider(IWorkspace workspace) => new MySettingsProvider(workspace);
}
```

>![TIP]
>When implementing methods that return `IWorkspaceSettingsSource` (like `IWorkspaceSettingsProvider.GetSingleSettings`), return an instance of `IWorkspaceSettings` rather than `IWorkspaceSettingsSource`. `IWorkspaceSettings` provides more information that can be useful during some settings aggregations.

### Related types and methods

- [IWorkspaceSettingsManager][Doc:IWorkspaceSettingsManager] reads and aggregates settings for workspace.
- [IWorkspace.GetSettingsManager][Doc:WorkspaceServiceHelper.GetSettingsManager] gets the `IWorkspaceSettingsManager` for a workspace.

## Detailed scenarios

### File watching

A workspace listens to file change notifications and provides the [IFileWatcherService][Doc:IFileWatcherService] via [IWorkspace.GetFileWatcherService][Doc:WorkspaceServiceHelper.GetFileWatcherService]. Events for items in a workspace's directory but filtered with the "ExcludedItems" setting will not generate file notification events. A threshold between events is used for notification simplification and duplicate reduction. When you need to react to a file change, you should subscribe to this service.

>![TIP]
>A workspace's indexing service is subscribed to file events. File addtions and modifications will cause relevant `IFileScanner`s to be invoked for new data for that file. File deletions will remove indexed data. You don't need to subscribe your `IFileScanner` to the file watcher service.

### Build support

Buzz words to include:

- BuildContextTypes
- IFileContextActionProvider
- IBuildConfigurationContext must be type of FileContext.Context

Up to date/incremental

#### Reporting build progress

`IFileContextAction.ExecuteAsync` will be supplied with the following types when invoked by Visual Studio:

- IProgress<[BuildMessage][Doc:BuildMessage]>
- [IBuildIncrementalProgressUpdate][Doc:IBuildIncrementalProgressUpdate]

### Debug support

Topics to include

- DebugLaunchActionContext
- IFileContextActionProvider

Set as default

File contexts

File data values

### Language service support

A language service might self-activate based on the file extension or content of an opened document, this "loose file" language service is limited to syntax highlighting. Additional info is required to provide a richer experience when editing/reviewing code. Each language service has its own API for initialization with this extra contextual data for a document. This is typically managed by a project system, which is tightly coupled both to the language service and to the build system.

In Open Folder, language services are initialized by an `WorkspaceSymbolSearchAsync` extension point that specializes only in that language service and knows nothing of the build authoring. In this way, a language service owner can maintain a single Open Folder extension regardless of how many patterns exist within folders and files for running their compiler during a build (e.g. MSBuild, makefiles, etc.). When files from which a file context was created are changed on disk and the file context is refreshed, the language service provider is notified of the updated set of file contexts. The language service provider can then update its model.

When a document is opened in the editor, Visual Studio only considers language service providers that require file context types for which a matching file context provider can be found. It then passes the file context(s) from the matching provider(s) to the selected language service provider via `ILangaugeServiceProvider.InitializeAsync`. What the language service provider does with that file context data is an implementation detail of the language service provider, but the expected user experience is a richer language service for that opened document.

#### How to make a language service

TO DO make example instead of this description

Define a Context class that describe the data asscociated to a file that will help a language service to consume it
Create a file context provider factory to produce File context that can be connected to your language service provider

Create a language service provider factory that is prepared to receive your context data of a file context and so something useful with it

#### Related interfaces

- [ILanguageServiceProvider][Doc:ILanguageServiceProvider] is invoked when a file of matching file types are opened or closed for editing.

## Suggested practices

- Return objects from `IWorkspaceProviderFactory.CreateProvider` or similar APIs that remember their `Workspace` context when created. Providers interfaces are written expecting this object is kept on creation.
- Save workspace-spcific caches or settings within the "Local settings" path of the workspace. Create a path for your file using [IWorkspace.MakeRootedUnderWorkingFolder][Doc:WorkspaceHelper.MakeRootedUnderWorkingFolder] in 15.6 or later. For prior to 15.6, use the following snippet:

```csharp
using System.IO;
using Microsoft.VisualStudio.Workspace;
using Microsoft.VisualStudio.Workspace.Settings;

private static string MakeRootedUnderWorkingFolder(IWorkspace workspace, string relativePath)
{
    string workingFolder = workspace.GetSettingsManager().GetAggregatedSettings(SettingsTypes.WorkspaceControlSettings).Property<string>("WorkingFolder");
    return Path.Combine(workingFolder, relativePath);
}
```

## Solution events and package auto-load

Loaded packages can implement `IVsSolutionEvents7` and invoke `IVsSolution.AdviseSolutionEvents`. It includes eventing on opening and closing a folder in Visual Studio.

A UI context can be used to to auto-load your package. The value is `4646B819-1AE0-4E79-97F4-8A8176FDD664`.

## Troubleshooting

### The 'SourceExplorerPackage' package did not load correctly

Workspace extensibility is heavily MEF based, and composition errors will cause the package hosting Open Folder to fail loading. For example, if an extension exports a type with `ExportFileContextProviderAttribute`, but the type only implements `IWorkspaceProviderFactory<IFileContextActionProvider>`. This will be seen when trying to open a folder in Visual Studio. If you see this while authoring your extension, then look at the errors in _%LOCALAPPDATA%\Microsoft\VisualStudio\15.0_id\ComponentModelCache\Microsoft.VisualStudio.Default.err_ . Resolve any errors for types implemented by your extension.

## Feedback, comments, issues

Open Folder and the `Microsoft.VisualStudio.Workspace.*` APIs are under active development. If you see unexpected behavior, then see the known issues for the release of interest. The Developer Community is the recommended place to vote and create any issues. For each feedback, we highly recommend a detailed description of the Visual Studio version you're developing for, the APIs you're using (both what you've implemented and services you're interacting with), the expected outcome, and the actual outcome. Use GitHub's issue tracking for giving feedback on this and related documentation.

[Doc:BuildContextTypes]:/dotnet/api/microsoft.visualstudio.workspace.build.buildcontexttypes
[Doc:BuildMessage]:/dotnet/api/microsoft.visualstudio.workspace.build.buildmessage
[Doc:ExportFileContextProviderAttribute]:/dotnet/api/microsoft.visualstudio.workspace.ExportFileContextProviderAttribute
[Doc:ExportFileContextActionProviderAttribute]:/dotnet/api/microsoft.visualstudio.workspace.exportfilecontextactionproviderattribute
[Doc:FileContext]:/dotnet/api/microsoft.visualstudio.workspace.filecontext
[Doc:IAsyncDisposable]:/dotnet/api/microsoft.visualstudio.threading.iasyncdisposable
[Doc:IBuildIncrementalProgressUpdate]:/dotnet/api/microsoft.visualstudio.workspace.build.ibuildincrementalprogressupdate
[Doc:IFileContextAction]:/dotnet/api/microsoft.visualstudio.workspace.ifilecontextaction
[Doc:IFileContextActionBase.ExecuteAsync]:/dotnet/api/microsoft.visualstudio.workspace.ifilecontextactionbase.executeasync
[Doc:IFileContextActionProvider]:/dotnet/api/microsoft.visualstudio.workspace.ifilecontextactionprovider
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
[Doc:IWorkspaceSettingsManager]:dotnet/api/microsoft.visualstudio.workspace.settings.iworkspacesettingsmanager
[Doc:WorkspaceHelper.GetWorkspaceWorkingFolder]:/dotnet/api/microsoft.visualstudio.workspace.workspacehelper.getworkspaceworkingfolder
[Doc:WorkspaceHelper.MakeRootedUnderWorkingFolder]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper.makerootedunderworkingfolder
[Doc:WorkspaceServiceHelper]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper
[Doc:WorkspaceServiceHelper.GetFileWatcherService]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper.getfilewatcherservice
[Doc:WorkspaceServiceHelper.GetServiceAsync]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper.getserviceasync
[Doc:WorkspaceServiceHelper.GetSettingsManager]:/dotnet/api/microsoft.visualstudio.workspace.workspaceservicehelper.getsettingsmanager
[Ex:FileScanner]:https://github.com/Microsoft/VSSDK-Extensibility-Samples/blob/master/Open_Folder_Extensibility/C%23/SymbolScannerSample/TxtFileSymbolScanner.cs
[Ref:TasksLaunchSettings]:/ide/customize-build-and-debug-tasks-in-visual-studio