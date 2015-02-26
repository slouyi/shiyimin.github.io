# 准备CLR源码阅读环境

微软发布了CLR 2.0的源码，这个源码是可以直接在freebsd和windows环境下编译及运行的，请在微软 [shared source cli](http://www.microsoft.com/en-us/download/details.aspx?id=4917) 链接处下载，并用7zip等工具解压，以后简称sscli – 即Shared Source CLI。

解压后，根目录下有readfirst.html文件，里面说明了该开源版本里包含的功能列表：

1. 泛型的实现；
1. 轻量级的代码生成；
1. 委托的实现；
1. 反射；
1. 装配件（Assembly）的元数据格式；
1. 匿名函数和委托；
1. .NET基本类库（BCL）；
1. 编译和运行代码

要编译clr源码的话，需要满足下面条件：

* Microsoft Visual Studio 2005专业版以上，请使用默认安装，避免在编译的时候各种找不到文件的情况；
* Perl。

另外，还有一个准备步骤，这个问题在中文版的Windows系统中都会遇到，由于Rotor中部分源代码以ANSI字符存放，其中中含有在936 Code Page，也就是Simplified Chinese GBK扩展字符集下无法解析的字符，在Build的时候VC编译器CL会报 ***warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss*** 。同时在Build的时候由于打开了/WX开关，任何warning都会被当作是error而直接导致部分编译失败。解决方法有：

1. 把全部有问题的源代码转换成Unicode编码；
1. 更改系统当前的区域设置（Locale）为英文

一般来说改系统的Locale最合适。在“控制面板”中“日期、时间、语言和区域设置”中的“区域和语言选项”的“高级”页中修改“非Unicode程序的语言”的选项为“英文（美国）” ，重启即可。
 
在编译过程中，需要将几个程序加入PATH路径，以便编译程序能找到它们：
 
将Perl的路径包含进来，如：`C:\Perl\bin`；

安装好上面的软件并将sscli源码解压之后，打开“Visual Studio 2005 Command Prompt”窗口，切换到sscli的根目录，下面假设根目录路径是：c:\sscli。依次执行下面的命令：

```
cd /d c:sscli
rem 设置当前编译和运行环境为调试环境
env debug
rem 编译所有的程序
buildall
```　

编译成功之后，写一个简单的C#文件，如下表：

```
using System;
 
public class Hello
{
      public static void Main()
      {
             Console.WriteLine("Hello, sscli");
      }
}
```　

在编译CLR源码的控制台运行下面的命令编译和执行C#程序（前面执行的 env debug命令，已经自动设置好PATH环境变量，C#编译器csc.exe程序指向的是我们编译好的程序）：

1. 编译：csc test.cs
1. 运行程序：clix test.exe

** 注意 **

1. 尽量使用英文原版的Windows XP进行编译，或者按前所述改成英文的区域设置；
1. 不要使用VS 2005以上的IDE编译，我曾经用VS 2008编译成功过，但写这篇文章的时候又碰到很多编译错误，为了省事的话，尽量用VS 2005编译；
1. 需要确认VS 2005安装好以后，有 `C:\Program Files\Microsoft Visual Studio 8\VC\PlatformSDK` 这个文件夹，在编译的时候，需要用到里面的头文件和库文件。