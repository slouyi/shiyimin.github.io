前文演示了编译SSCLI最简便的方法（在Windows下）：

 1. 在“Visual Studio 2005 Command Prompt”下，进入SSCLI的根目录；
 2. 运行 env.bat 脚本准备环境；
 2. 运行 buildall.cmd 脚本开始编译过程。

env.bat设置了当前SSCLI的运行环境，命令的语法是：`env [option]`，其中[option]可以是 *debug*、*checked* 和 *free* ，各个环境选项说明如下表：

* **debug**：关闭代码优化设置，启用调试用代码（一般是通过条件编译开关），生成调试用符号文件。
* **checked**： 打开代码优化设置，启动调试用代码，生成符号文件。
* **free** ：打开代码优化设置，关闭调试用代码，生成符号文件。
出于解读代码的需要，一般都会选择debug选项。

设定好运行环境后，buildall.cmd的编译过程可以分为下面几大步骤：

1. 编译基本的引导工具集：平台抽象层（PAL）和非托管程序（unmanaged）编译工具；
1. 编译辅助的引导工具集：剩余的工具集和相关基础架构；
1. 编译CLR内核、基本类库，C#编译器和其他支持工具；
1. 编译其它.NET类库和支持工具
1. 编译其它托管程序的编译器，如JS编译器。

其实.NET天生就是跨平台的，SSCLI 2.0除了支持Windows平台，还支持FreeBSD和Mac OS X，源码可以在FreeBSD 4.8和Mac OS X 10.2下编译通过，通过修改一些源码，还可以在Linux平台下编译和使用，实际上SSCLI 1.1通过一些修改可以在Red Hat 8.0下编译通过。

为了在后文说明方便，这里用SSCLI内置的几个环境变量来指代将会提到的文件夹路径：

* **%ROTOR_DIR%**： SSCLI的根目录，如Windows平台下一般是`c:\sscli`，在编译过程中，编译系统会自行设置此环境变量。
* **%_NTTREE%**：整个SSCLI编译输出结果的保存路径，其值一般是：`%ROTOR_DIR%\binaries.x86*.rotor\`，*号根据bat 的选项赋值，如env.bat的选项是`debug`，那么它的值就是 `%ROTOR_DIR%\binaries.x86dbg.rotor\`。

基本引导工具集
--------------------
SSCLI的源码由C++、C#这些非托管和托管编程语言组成，而且SSCLI是跨平台的，因此在编译过程中编译系统会用操作系统上安装的C++编译器编译好编译SSCLI剩余源码的工具集。在这一步编译过程中，基本引导工具集包括下面这些组件：

* 平台抽象层（PAL）：PAL是系统中其它所有组件的依赖项，因此必须先编译它：

   * 源码位置（Windows平台）：%ROTOR_DIR%\pal\win32
   * 源码位置（Unix平台）：%ROTOR_DIR%\pal\unix
   * 输出路径：%_NTTREE%\rotor_pal.dll

* Nmake工具集：nmake是Windows SDK系统下的编译工具，类似unix下的make。SSCLI自带了nmake的源码，如果是在Unix或Linux系统下编译，会编译此工具；或者在Windows平台下，就用SDK系统下自带的nmake工具了：
  * 源码位置：%ROTOR_DIR%\tools\nmake
  * 输出路径：%_NTTREE%\nmake.exe

* 编译结果的处理工具：
  * 源码位置：%ROTOR_DIR%\tools\binplace
  * 输出路径：%_NTTREE%\binplace.exe

* 编译系统其它组件的编译工具集：
  * 源码位置：%ROTOR_DIR%\tools\build
  * 输出路径：%_NTTREE%\build.exe

辅助引导工具集
--------------------
辅助引导工具集与前面基本引导工具集的区别是，辅助引导工具集用的是SSCLI编译系统自带的build.exe编译的，源文件列表等编译规则是保存在sources.lst文件里的，而基本引导工具集是用make命令编译，而且源文件列表等编译规则是保存在makefile文件夹里的。

辅助引导工具集包括下列这些组件：

* 资源编译器
   * 源码位置：%ROTOR_DIR%\tools\resourcecompiler
   * 输出路径：%_NTTREE%\resourcecompiler.exe

* PAL运行时（PAL RT）：PAL RT包含了SSCLI和其一些辅助工具都用到的跨平台的功能。

  * 源码位置：%ROTOR_DIR%\palrt\src
  * 输出路径：%_NTTREE%\rotor_palrt.dll

编译CLR内核、基本类库，C#编译器和其他支持工具
--------------------------------------------------------------

引导工具集编译完成后，就可以开始编译整个CLR、.NET基本类库、C#编译器等工具了，各工具的源码位置、输出路径如下表所示：

* **\clr\src**：	整个CLR内核和基本类库的源码路经
* **\clr\src\vm**：CLR虚拟机引擎的源码，包括GC、JIT、类库加载器等完整功能
* **\csharp**：C#编译器和Assembly链接程序的源码路经
* **\clr\src\bcl**：.NET基本类库源码，如System.IO、System.Collections这些命名空间下面的类库源码都在这里
* **\clr\src\dlls**：	其它进程跟CLR虚拟机交换的几个关键DLL的源码，包括下面这些组件：
* **shim**：主要是确定当前机器上安装了哪些.net版本
* **mscorsn**：用来做强签名验证
* **mscorpe**：Windows平台可执行文件PE格式读写库
* **mscoree**：用来在进程中加载CLR虚拟机
* **mscordbi和mscordbc**：调试服务
* **\src\utilcode**：被很多CLR虚拟机组件共享的通用代码
* **\src\fjit**：JIT编译器，即mscorejit.dll的源码
* **\src\fusion**：搜索和加载Assembly、GAC（Global Assembly Cache）的组件，即fusion.dll的源码
* **\src\ilasm**：IL语言编译器
* **\src\ildasm**：IL反编译工具
* **\src\debug**：托管调试器 cordbg 的源码
* **\src\md**：Assembly元数据读写库
* **\src\tools**：多个非托管工具的源码：
* **clix**：托管程序解释执行工具
* **ildbsymbols**：托管程序调试符号文件读写工具
* **metainfo**：assembly元数据读取工具
* **peverify**：验证托管程序的IL程序
* **internalresgen**：
* **sn**：强签名操作工具
* **permview**：Assembly权限查看工具
* **gac**：全局Assembly缓存管理工具
* **sos**：Windbg调试.NET程序的辅助函数库

其它.NET类库和支持工具
-------------------------------
这里面提供了一些除基本类库以外的少量类库源码：

* **System.dll**：%ROTOR_DIR%\fx\src\sys
* **System.xml.dll**：%ROTOR_DIR%\fx\src\xml
* **System.Runtime.Serialization.Formatters.Soap.dll**：%ROTOR_DIR%\managedlibraries\soapserializer
* **System.Runtime.Remoting.dll**：%ROTOR_DIR%\managedlibraries\remoting

其它托管程序的编译器
----------------------------

SSCLI里面还附带了一个使用C#语言开发的Microsoft Jscript编译器实现，而这个编译器本身就能编译托管程序，其源码路经是：`%ROTOR_DIR%\jscript`