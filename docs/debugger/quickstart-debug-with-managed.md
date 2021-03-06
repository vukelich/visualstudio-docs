---
title: "Debug with managed code using the Visual Studio debugger | Microsoft Docs"
ms.custom: ""
ms.date: "12/06/2017"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "vs-ide-debug"
ms.tgt_pltfrm: ""
ms.topic: "quickstart"
helpviewer_keywords: 
  - "debugger"
ms.assetid: f4cea2e1-08dc-47ac-aba2-3b8c338e607f
caps.latest.revision: 1
author: "mikejo5000"
ms.author: "mikejo"
manager: ghogen
ms.workload: 
  - "dotnet"
---
# Debug with managed code using the Visual Studio debugger

The Visual Studio debugger provides many powerful features to help you debug your apps. This topic provides a quick way to learn some of the basic features.

## Create a new project 

1. In Visual Studio, choose **File > New Project**.

2. Under **Visual C#** or **Visual Basic**, choose **.NET Core**, and then in the middle pane choose **Console App (.NET Core)**.

     If you don't see the **Console App (.NET Core)** project template, click the **Open Visual Studio Installer** link in the left pane of the **New Project** dialog box. The Visual Studio Installer launches. Choose the **.NET desktop development** and **.NET Core** workload, then choose **Modify**.

3. Type a name like **MyDbgApp** and click **OK**.

    Visual Studio creates the project.

4. In Program.cs or Module1.vb, replace the following code

    ```c#
    class Program
    {
        static void Main(string[] args)
        {
        }
    }
    ```

    ```vb
    Module Module1
        Sub Main()
        End Sub
    End Module
    ```

    with this code:

    ```c#
    class Program
    {
        private static void doWork()
        {
            LinkedList<int> c1 = new LinkedList<int>();

            c1.AddLast(10);
            c1.AddLast(20);

            LinkedList<int> c2 = new LinkedList<int>(c1);
            int i = c2.First.Value;
            int j = c2.First.Value;
            Console.Write("The first element is ");
            Console.Write(i);
            Console.Write("\n");
            Console.Write("The second element is ");
            Console.Write(j);
            Console.Write("\n");

        }

        static int Main()
        {
            // using namespace std;
            doWork();
            return 0;

        }
    }
    ```

    ```vb
    Imports System.Collections.Generic

    Namespace MyDbgApp
        Class Program
            Private Shared Sub doWork()
                Dim c1 As New LinkedList(Of Integer)()

                c1.AddLast(10)
                c1.AddLast(20)

                Dim c2 As New LinkedList(Of Integer)(c1)
                Dim i As Integer = c2.First.Value
                Dim j As Integer = c2.First.Value
                Console.Write("The first element is ")
                Console.Write(i)
                Console.Write(vbLf)
                Console.Write("The second element is ")
                Console.Write(j)
                Console.Write(vbLf)

            End Sub

            Public Shared Function Main() As Integer
                ' using namespace std;
                doWork()
                Return 0

            End Function
        End Class
    End Namespace
    ```

    > [!NOTE]
    > In Visual Basic, make sure the startup object is set to `Sub Main` (**Properties > Application > Startup Object**).

## Set a breakpoint

A *breakpoint* is a marker that indicates where Visual Studio should suspend your running code so you can take a look at the values of variables, or the behavior of memory, or whether or not a branch of code is getting run. It is the most basic feature in debugging.

1. To set the breakpoint, click in the gutter to the left of the `doWork` function call (or select the line of code and press **F9**).

    ![Set a breakpoint](../debugger/media/dbg-qs-set-breakpoint-csharp.png "Set a breakpoint")

2. Now press **F5** (or choose **Debug > Start Debugging**).

    ![Hit a breakpoint](../debugger/media/dbg-qs-hit-breakpoint-csharp.png "Hit a breakpoint")

    The debugger pauses where you set the breakpoint. The statement where the debugger and app execution is paused is indicated by the yellow arrow. The line with the `doWork` function call has not yet executed.

    > [!TIP]
    > If you have a breakpoint in a loop or recursion, or if you have a lot of breakpoints which you frequently step through, use a [conditional breakpoint](../debugger/using-breakpoints.md#BKMK_Specify_a_breakpoint_condition_using_a_code_expression) to make sure that your code is suspended ONLY when specific conditions are met. This saves time and can also make it easier to debug issues that are hard to reproduce.

## Navigate code

There are different commands to instruct the debugger to continue. We will show a useful code navigation command that is new in Visual Studio 2017.

- While paused at the breakpoint, hover over the statement `c1.AddLast(20)` until the green **Run to click** button ![Run to Click](../debugger/media/dbg-tour-run-to-click.png "RunToClick") appears, and then press the **Run to click** button.

    ![Run to click](../debugger/media/dbg-qs-run-to-click-csharp.png "Run to click")

    The app continues execution, calling `doWork`, and pauses on the line of code where you clicked the button.

    Common keyboard commands used to step through code include **F10** and **F11**. For more in-depth instructions, see the [Beginner's Guide](../debugger/getting-started-with-the-debugger.md).

## Inspect variables in a datatip

1. In the current line of code (marked by the yellow execution pointer), hover over the `c1` object with your mouse to show a datatip.

    ![View a datatip](../debugger/media/dbg-qs-data-tip-csharp.png "View a datatip")

    The datatip shows you the current value of the `c1` variable and allows you to inspect its properties. When debugging, if you see a value you don't expect, you probably have a bug in the preceding or calling lines of code. 

2. Expand the datatip to look at the current property values of the `c1` object.

3. If you want to pin the datatip so that you can continue to see the value of `c1` while you execute code, click the small pin icon. (You can move the pinned datatip to a convenient location.)

## Edit code and continue debugging

If you identify a change that you want to test in your code while in the middle of a debugging session, you can do that, too.

1. Click the second instance of `c2.First.Value` and change `c2.First.Value` to `c2.Last.Value`.

2. Press **F10** (or **Debug > Step Over**) a few times to advance the debugger and execute the edited code.

    ![Edit and continue](../debugger/media/dbg-qs-edit-and-continue-csharp.gif "Edit and continue")

    **F10** advances the debugger one statement at a time, but steps over functions instead of stepping into them (the code that you skip still executes).

For more information on using edit-and-continue and on feature limitations, see [Edit and Continue](../debugger/edit-and-continue.md).

## Next steps

- To learn more about the debugger, see [Start the debugger and navigate code](../debugger/getting-started-with-the-debugger.md).
- To find out more about breakpoints, see [Using breakpoints](../debugger/using-breakpoints.md).

## See Also  
 [Debugging in Visual Studio](../debugger/index.md)  
 [Debugger Feature Tour](../debugger/debugger-feature-tour.md)
