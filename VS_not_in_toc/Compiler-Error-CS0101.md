---
title: "Compiler Error CS0101"
ms.custom: na
ms.date: 10/01/2016
ms.devlang: 
  - CSharp
ms.prod: visual-studio-dev14
ms.reviewer: na
ms.suite: na
ms.technology: 
  - devlang-csharp
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: edb5246b-c16b-4845-bb2d-0ef769d014c7
caps.latest.revision: 8
manager: douge
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
# Compiler Error CS0101
The namespace 'namespace' already contains a definition for 'type'  
  
 A [namespace](../Topic/namespace%20\(C%23%20Reference\).md) has duplicate identifiers. Rename or delete one of the duplicate identifiers. For more information, see [Namespaces](../Topic/Namespaces%20\(C%23%20Programming%20Guide\).md)  
  
 The following sample generates CS0101:  
  
```  
// CS0101.cs  
namespace MyNamespace  
{  
   public class MyClass  
   {  
      static public void Main()  
      {  
      }  
   }  
  
   public class MyClass   // CS0101  
   {  
   }  
}  
```