阅读源码一个比较快的手段就是在调试器里阅读，这样可以在实际运行SSCLI的过程中，通过堆栈跟踪的方式查看完整的程序执行路径。

当在SSCLI环境里执行一个托管程序的时候，堆栈上通常有托管和非托管代码同时在执行。因此在SSCLI里也支持下面几种调试场景：

* **调试托管程序**：在SSCLI里自带了一个托管调试程序，cordbg.exe。跟调试普通.net程序不同，目前还无法在Visual Studio里调试SSCLI环境下的托管程序，在后面有时间的时候，我们会看一下如何在Visual Studio里添加调试SSCLI托管程序的支持。
* **调试非托管程序**：即调试SSCLI虚拟机本身的非托管代码，可以使用Visual Studio、WinDbg等普通Windows调试工具进行调试。在非Windows平台下，也可以用gdb这些工具进行调试。
* **托管非托管程序混合调试**：可以Visual Studio和Windbg这些Windows调试工具下通过加载sos调试插件来实现混合调试的功能。另外，sscli也提供了sos调试插件的源码。

SSCLI里的符号文件
------------------------
在SSCLI里，会生成两种符号文件，一种是编译非托管程序生成的.pdb文件，一种是编译托管程序生成的.ildb文件。关于符号文件的作用，请参看我之前的文章：[Visual Studio调试之符号文件](http://www.cnblogs.com/killmyday/archive/2009/09/26/1574311.html)。

Windows调试工具调试都支持.pdb文件，而cordbg.exe可以识别.ildb格式，而sos插件不需要任何符号文件即可调试。

SSCLI提供了工具ildbconv程序可以在.pdb文件和.ildb文件格式之间相互转换，因此如果要在Visual Studio里调试SSCLI里编译的托管程序，可以用这个工具将对应的.ildb文件转换成Visual Studio支持的pdb格式，并在.NET环境下运行托管程序即可调试。ildbconv命令的格式如下：

> ildbconv [选项] 符号文件路径

其中ildbconv支持下面这些选项：

* **/toildb**：将.pdb文件转换成.ildb格式；
* **/topdb**：将.ildb文件转换成.pdb格式。

SSCLI也提供了ildbconv程序的源码，其路径位于：`\tools\ildbconv\ildbconv.cpp`