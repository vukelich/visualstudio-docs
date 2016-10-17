---
title: "Toolbox, Components Tab"
ms.custom: na
ms.date: 10/10/2016
ms.prod: visual-studio-dev14
ms.reviewer: na
ms.suite: na
ms.technology: 
  - vs-ide-general
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 332fafab-a763-4244-b388-15d1b5b5cc04
caps.latest.revision: 14
manager: ghogen
translation.priority.ht: 
  - cs-cz
  - de-de
  - es-es
  - fr-fr
  - it-it
  - ja-jp
  - ko-kr
  - pl-pl
  - pt-br
  - ru-ru
  - tr-tr
  - zh-cn
  - zh-tw
---
# Toolbox, Components Tab
Displays components you can add to Visual Basic and Visual C# designers. In addition to the .NET Framework components that are included with Visual Studio, such as the <xref:System.Messaging.MessageQueue?qualifyHint=False> and <xref:System.Diagnostics.EventLog?qualifyHint=False> components, you can add your own or third-party components to this tab. For more information, see [How to: Manipulate Toolbox Tabs](assetId:///21285050-cadd-455a-b1f5-a2289a89c4db).  
  
 To display this tab, from the **View** menu, select **Toolbox**. In the **Toolbox**, select the **Components** tab.  
  
 **BackgroundWorker**  
 Creates a `System.ComponentModel.BackgroundWorker` component instance that can run an operation on a separate, dedicated thread.  
  
 **DirectoryEntry**  
 Creates a <xref:System.DirectoryServices.DirectoryEntry?qualifyHint=False> component instance that encapsulates a node or object in the Active Directory hierarchy and can be used to interact with Active Directory service providers.  
  
 **DirectorySearcher**  
 Creates a <xref:System.DirectoryServices.DirectorySearcher?qualifyHint=False> component instance that you can use to perform queries against the Active Directory.  
  
 **ErrorProvider**  
 Creates a `System.Windows.Forms.ErrorProvider` component instance, which indicates to the end user that a control on a form has an error associated with it.  
  
 **EventLog**  
 Creates an <xref:System.Diagnostics.EventLog?qualifyHint=False> component instance you can use to interact with system and custom event logs, including writing events to a log and reading log data. For more information, see [Introduction to the EventLog Component](assetId:///a2ba4f28-4b1a-435e-99ef-51b28e21f805).  
  
 **FileSystemWatcher**  
 Creates a <xref:System.IO.FileSystemWatcher?qualifyHint=False> component instance that you can use to monitor for changes to any directory or file to which you have access. For more information, see [How to: Configure FileSystemWatcher Component Instances](assetId:///2e628234-4951-4135-8a86-28b924070d50).  
  
 **HelpProvider**  
 Creates a `System.Windows.Forms.HelpProvider` component instance that provides pop-up or online Help for controls.  
  
 **ImageList**  
 Creates a `System.Windows.Forms.ImageList` component instance that provides methods to manage a collection of `System.Drawing.Image` objects.  
  
 **MessageQueue**  
 Creates a <xref:System.Messaging.MessageQueue?qualifyHint=False> component instance that you can use to interact with message queues, including reading messages from and writing messages to queues, processing transactions, and performing queue administration tasks. For more information, see [Using Messaging Components](assetId:///922dbac7-26f0-4e39-b666-ccfc184793d7).  
  
 **PerformanceCounter**  
 Creates a <xref:System.Diagnostics.PerformanceCounter?qualifyHint=False> component instance that you can use to interact with Windows performance counters, including creating new categories and instances, reading values from counters, and performing calculations on counter data. For more information, see [Monitoring Performance Thresholds](assetId:///b8b44a55-31d0-4b45-9517-8c1b1e4fdc91).  
  
 **Process**  
 Creates a <xref:System.Diagnostics.Process?qualifyHint=False> component instance you can use to stop, start, and manipulate the data associated with processes on your system. For more information, see [Monitoring and Managing Windows Processes](assetId:///a86bd4c1-b92c-49a0-8f32-61d67837b45e).  
  
 **SerialPort**  
 Creates a `System.IO.Ports.SerialPort` component instance that provides synchronous and event-driven I/O, access to pin and break states, and access to serial driver properties.  
  
 **ServiceController**  
 Creates a <xref:System.ServiceProcess.ServiceController?qualifyHint=False> component instance you can use to manipulate existing services, including starting and stopping services and sending commands to them. For more information, see [Monitoring Windows Services](assetId:///4542ee3f-e052-4cb9-8726-58e9420de222).  
  
 **Timer**  
 Creates a <xref:System.Windows.Forms.Timer?qualifyHint=False> component instance you can use to add time-based functionality to your Windows-based applications. For more information, see [Timer Component](../Topic/Timer%20Component%20\(Windows%20Forms\).md).  
  
> [!NOTE]
>  There is also a system-based <xref:System.Timers.Timer?qualifyHint=False> that you can add to the **Toolbox** This <xref:System.Timers.Timer?qualifyHint=False> is optimized for server applications, and the Windows Forms <xref:System.Windows.Forms.Timer?qualifyHint=False> is best suited for use on Windows Forms.  
  
## See Also  
 [Programming with Components](../Topic/Programming%20with%20Components.md)   
 [Component Programming Walkthroughs](../Topic/Component%20Programming%20Walkthroughs.md)   
 [Toolbox](../VS_IDE/Toolbox.md)   
 [Choose Toolbox Items Dialog Box (Visual Studio)](assetId:///bd07835f-18a8-433e-bccc-7141f65263bb)