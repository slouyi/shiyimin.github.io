# 在SSCLI中调试托管代码

SSCLI只实现了部分.NET调试功能，而且不支持在Visual Studio里直接调试SSCLI环境里执行的托管程序，在SSCLI里只能用其自带的cordbg.exe调试SSCLI环境下的托管程序。

因为.NET Framework 2.0 SDK也自带了cordbg.exe，所以需要在SSCLI环境里（即执行了 `env.bat` 命令初始化过的SSCLI），否则会采用错误的cordbg版本调试SSCLI程序。

我们将用下面这个C#程序来演示cordbg的用法，其是SSCLI自带的例子，源码位于：`samples\howto\basedatatypes\stringformat`：

```C#
using System;
using System.IO;
using System.Globalization;

public enum Color
{
    Red = 25,
    Blue,
    Green
}//enum Color

public class stringFormat
{
    public static void Main(string[] args)
    {
        DateTime d = DateTime.Now;

        PrintFormat(d, "d");

        Console.WriteLine("AbbreviatedDayNames = new string [] {Sund, Mond, Tues, Weds, Thur, Frid, Satu}");
        DateTimeFormatInfo dtfi = new DateTimeFormatInfo();
        dtfi.AbbreviatedDayNames =
                   new string[] { "Sund", "Mond", "Tues", "Weds", "Thur", "Frid", "Satu" };
        PrintFormat(d, "ddd", dtfi);
        Console.WriteLine("------------------------------------------------------------------------------\n");

        // Enum formatting
        Console.WriteLine("Enum Formatting");
        Console.WriteLine
          (
          "Name: {0}, Enum Value: {1}, Enum Value Hex: {2}",
          Color.Green.ToString("G"),
          Color.Green.ToString("D"),
          Color.Green.ToString("X")
          );
        Console.WriteLine();

        //Number standard formats
        double num = System.Math.PI * -3.0;
        PrintFormat(num, "c4");
    } //Main()

    public static void PrintFormat(IFormattable inputData, string formatString)
    {
        PrintFormat(inputData, formatString, null);
    }//PrintFormat()

    public static void PrintFormat(IFormattable inputData,
                                   string formatString,
                                   IFormatProvider provider)
    {
        try
        {
            if (provider == null)
            {
                Console.WriteLine("{0}\t{1}",
                                  formatString,
                                  inputData.ToString(formatString, provider));
            }//if
            else
            {
                string formstr = "{0} in the custom " + formatString + " format is {1:" + formatString + "}";
                Console.WriteLine(string.Format(provider, formstr, inputData, inputData));
            }//else
        }//try
        catch (Exception e)
        {
            Console.WriteLine("Exception in PrintFormat(): {0}", e.Message);
            Console.WriteLine("Data was {0}, Format string was: {1}, Type was {2}",
                               inputData,
                               formatString,
                               inputData.GetType().Name);
        }//catch
    }//PrintFormat
}//class stringformat
```

编译的时候，需要打开C#编译器的“/debug”调试开关才能生成调试版本的程序，便于调试器（cordbg）调试程序：

`csc /debug stringformat.cs`

注意：csc命令也要在初始化SSCLI环境之后才能运行，否则也会用到.NET Framework SDK自带的csc命令。

命令执行完毕之后，会看到编译器除了输出stringformat.exe可执行文件以外，还生成了stringformat.ildb这个调试用的符号文件。

执行下面的命令开始调试：

`cordbg stringformat.exe`

cordbg运行界面如下图：

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter1/cordbg-interface-overview.png)

与在Visual Studio这些IDE调试程序不同，cordbg一开始并不会直接运行被调试程序，而是先中断程序的执行 – 即在第15行Main函数的入口处中断，等待命令输入。这是因为需要留出时间让开发人员准备调试工作 – 如设置断点之类的。在命令行里输入“help”命令可以查看cordbg里可用的命令，下一篇文章将讲解cordbg的命令用法。

这里我们演示几个基本的命令，下表我用批注（以#号开头的粗体字体）说明每个命令的意义：

```bash
C:\sscli20\research\chapter1>cordbg stringformat.exe
Microsoft (R) Common Language Runtime Test Debugger Shell Version 2.0.50826.0
Copyright (c) Microsoft Corporation.  All rights reserved.

#
# run 命令用来启动被调试程序，如果执行cordbg命令时，传入了被调试程序文件的参数，
# 则会自动执行这个命令
#

(cordbg) run stringformat.exe
Process 1588/0x634 created.
[thread 0xe4] Thread created.

#
# 当前程序中断的位置，前面的 015 表示当前中断的位置是第15行
#

015:     {

#
# 在源码的第18行设置断点
#

(cordbg) break 18
Breakpoint #1 has bound to C:\sscli20\research\chapter1\stringformat.exe.
#1      c:\sscli20\research\chapter1\stringformat.cs:18 Main+0x7(il) [active]

#
# 恢复程序的执行
#

(cordbg) g
break at #1     c:\sscli20\research\chapter1\stringformat.cs:18 Main+0x7(il) [ac
tive]

#
# 触发了断点并中断执行程序
#

018:         PrintFormat(d, "d");

#
# 查看局部变量d的值
#

(cordbg) p d
d=<System.DateTime>
  dateData=0x88d21f5af1505bc5
  
#
# 查看所有函数参数、局部变量和全局变量
#

(cordbg) p
args=(0x00d4217c) array with dims=[0]
CS$0$0000=<null>
d=<System.DateTime>
dtfi=<null>
num=0
$result=(strong handle, Size: 4) (0x00d43fe0) <System.Reflection.TargetParameter
CountException>
$thread=(0x00d42830) <System.Threading.Thread>

#
# 调用一个函数，需要给出函数所属类型的全名 – 即包括命名空间，
# 类名和函数名之间用“::”分隔，参数通过空格分隔
#

(cordbg) funceval System.DateTime::ToString d
There are 4 possible matches for the method ToString. Pick one:

#
# 因为有函数重载的关系，所以cordbg会列出可选的函数，并让开发人员指定
# 所要调用的函数。
#

0) none, abort the operation.
1) [060002eb] String ToString()
2) [060002ec] String ToString(String)
3) [060002ed] String ToString(IFormatProvider)
4) [060002ee] String ToString(String, IFormatProvider)

Please make a selection (0-4): 1
Function evaluation complete.
$result=(strong handle, Size: 4) (0x00d5b6a0) "2015/2/25 21:41:26"
018:         PrintFormat(d, "d");

#
# funceval调用函数的结果被保存到一个叫 $result 的变量里，在执行后续的funceval
# 命令时，可以重用上一个命令的调用结果
#

(cordbg) funceval System.Console::WriteLine $result
There are 19 possible matches for the method WriteLine. Pick one:
0) none, abort the operation.
1) [060006d4] Void WriteLine()
... ...
14) [060006e1] Void WriteLine(String)
... ...
19) [060006e6] Void WriteLine(String, Object[])

Please make a selection (0-19): 14
2015/2/25 21:41:26
Function evaluation complete, no result.

018:         PrintFormat(d, "d");

#
# k命令用来终止当前正在执行的被调试进程
#

(cordbg) k
Terminating current process...
Process exited.

#
# exit命令退出cordbg程序
#

(cordbg) exit
```