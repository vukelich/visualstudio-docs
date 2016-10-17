---
title: "DA0014: Extremely high rates of paging active memory to disk"
ms.custom: na
ms.date: 10/03/2016
ms.prod: visual-studio-dev14
ms.reviewer: na
ms.suite: na
ms.technology: 
  - vs-ide-debug
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: a7fa3749-9191-437a-9331-9d917181e62f
caps.latest.revision: 11
manager: ghogen
translation.priority.ht: 
  - de-de
  - es-es
  - fr-fr
  - it-it
  - ja-jp
  - ko-kr
  - ru-ru
  - zh-cn
  - zh-tw
translation.priority.mt: 
  - cs-cz
  - pl-pl
  - pt-br
  - tr-tr
---
# DA0014: Extremely high rates of paging active memory to disk
|||  
|-|-|  
|Rule Id|DA0014|  
|Category|Memory and Paging|  
|Profiling method|All|  
|Message|An extremely high rate of paging active memory to disk is occurring. Your application may be memory-bound.|  
|Rule type|Warning|  
  
 When you profile by using the sampling, .NET memory, or resource contention methods, you must collect at least 25 samples to trigger this rule.  
  
## Cause  
 System performance data that was collected in the profiling run indicates that an extremely high rate of paging active memory to and from the disk occurred throughout the profiling run. Paging rates at this level usually impacts application performance and responsiveness. Consider reducing memory allocations by revising algorithms. You might also have to consider the memory requirements of your application. running profiling again on a computer with more memory.  
  
## Rule Description  
 Excessive paging to disk can be caused by a shortage of physical memory. If paging operations dominate use of the physical disk where the paging file resides, they can slow down other application-oriented disk operations to the same disk.  
  
 Frequently, pages are read from the disk or written to the disk in bulk paging operations. The number of Pages Output/sec is frequently much larger than the number of Page Writes/sec, for example. Because Pages Output/sec also includes changed data pages from the system file cache. However, it is not always easy to determine which process is directly responsible for the paging or why.  
  
> [!NOTE]
>  This rule fires when levels of paging of active memory reach a very high rate. When the level of paging is significant, but not extreme, the informational rule [DA0017: High rates of paging active memory to disk](../VS_IDE/DA0017--High-rates-of-paging-active-memory-to-disk.md) fires instead.  
  
## How to Fix Violations  
 Double-click the message in the Error List window to navigate to the [Marks](../VS_IDE/Marks-View.md) view. Find the **Memory\Pages/sec** column. Determine if there are specific phases of program execution where paging IO activity is heavier than others.  
  
 If you are gathering profile data for an ASP.NET application in a load testing scenario, try running the load test again on a machine configured with additional physical memory (or RAM).  
  
 Consider reducing memory allocations by revising algorithms and avoiding memory-intensive APIs such as String.Concat and String.Substring.