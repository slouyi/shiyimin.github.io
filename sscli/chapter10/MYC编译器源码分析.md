源码分析
----

整个编译器是用C#语言写的，上图列出了MyC编译器编译一个C源文件的过程，编译主路径如下：

1.	首先是入口Main函数用来解析命令行参数，读取源文件，并开始编译过程。Main函数在MyC.cs文件，而IO.cs文件主要保存读取源码文件的相关操作。下表是Main函数的源码（批注用注释的方式显示），IO.cs文件用单独的一个小节说明：

``` CSharp 
public static void Main()
{
    try
{
　// 看源码注释，代码是99年写的，也就是说.NET当年正在开发中
    // 可能那个时候虚拟机都没有做好向Main函数传递命令行参数的开发
    // 用了下面这个奇葩方法获取程序的命令行参数
        String[] args = Environment.GetCommandLineArgs();
        // 初始化读取源文件的IO对象，该对象负责将源文件以字节流的方式
        // 输出给下一个对象 – 词法分析
        Io prog = new Io(args);
        // 词法分析对象，该对象的工作是过滤掉源码中不必要的字符，比如空格
        // 注释之类的，并且把源码中的字符归类 – Tokenize，以便语法分析器
        // 更方便的解析语法
        Tok tok = new Tok(prog);
        // 语法分析对象，解析完毕后即是代码生成阶段，但一般语法分析过程
        // 都只会生成语法树，这样的设计可以对接多种结果文件输出手段。比如
        // 说，本例中生成可执行文件的exe.cs和生成IL源码的asm.cs都是通过
        // 遍历语法树，使用不同的输出策略生成结果文件的
        Parse p = new Parse(prog, tok);
        // 采用自顶向下的方式进行语法解析
        p.program();
        // 编译工作已经完成，关闭打开的源文件句柄等资源
        prog.Finish();
    }
    catch (Exception e)
{
    // 编译过程中有任何错误，即中断处理，打印错误消息并退出程序
        Console.WriteLine("Compiler aborting: " + e.ToString());
    }
}
```

2.	MyC的语法很简单，因此编译过程是很干净的词法分析、语法分析、代码生成和结果输出的过程。其中词法分析代码在tok.cs，语法分析代码在parse.cs中，Emit.cs处理代码生成，而Asm.cs和Exe.cs分别根据命令行参数的设置，来生成最终的可执行文件并选择是否输出IL源码。
IO.cs - IO处理
将源文件读取进内存，并采用流式处理的代码都放在IO这个类里面，IO的构造函数解析命令行参数，并打开源文件，等待Tok.cs里面代码的指令将源文件的字符一个个读进内存并处理，下面是它的构造函数的源码：

``` CSharp
public Io(String[] a)
  {
  int i;

  args = a;
  // 解析命令行参数，并根据参数打开内部的控制开关，详情请看下面对ParseArgs
  // 函数的源码解读
  ParseArgs();

  // 打开要编译的源文件
  ifile = new FileStream(ifilename, FileMode.Open,
			 FileAccess.Read, FileShare.Read, 8192);
  // 如果源文件不存在，报错退出
  if (ifile == null)
    {
    Abort("Could not open file '"+ifilename+"'\n");
}
  // 采用流式处理方式读取源文件
  rfile = new StreamReader(ifile); // open up a stream for reading
 
  // 根据源文件的名称设定结果输出文件的文件名
  i = ifilename.LastIndexOf('.');
  if (i < 0)
    Abort("Bad filename '"+ifilename+"'");
  int j = ifilename.LastIndexOf('\\');
  if (j < 0)
    j = 0;
  else
    j++;

  classname = ifilename.Substring(j,i-j);

  // 根据命令行参数决定是生成.exe、.dll等可执行文件，还是输出包含
  // IL源码的.lst文件 
  if (genexe)
    ofilename = classname+".exe";
  if (gendll)
    ofilename = classname+".dll";
  if (genlist)
{
// 如果是要输出IL源码，因为原来的可执行文件也要输出，需要创建一个新的文件
    lst_ofilename = classname+".lst";
    lst_ofile = new FileStream(lst_ofilename, FileMode.Create,
			 FileAccess.Write, FileShare.Write, 8192);
    if (lst_ofile == null)
      Abort("Could not open file '"+ofilename+"'\n");
    lst_wfile = new StreamWriter(lst_ofile);
    }
  }
```

编译器是在IO类里处理命令行参数的，参数解析实际上是一些字符串处理的活，本文解释下关键代码：

``` CSharp
void ParseArgs()
  {
  int i = 1;
  
  // 程序至少需要两个参数，否则就输出帮助文字并退出
  if (args.Length < 2)
    {
    Abort("myc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
    }
  
  // 逐个遍历命令行参数
  while (true)
    {
    if (args[i][0] != '/')
      break;
    // 处理 /? 这个参数，即输出帮助文本
    if (args[i].Equals("/?"))
      {
      Console.WriteLine("Compiler options:\n  myc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
      Environment.Exit(1);
      }
// 如果有 /debug 参数，则打开内部的 gendebug 开关，这个开关在代码生成的过程
// 中会用到
    if (args[i].Equals("/debug"))
      {
      gendebug = true;
      i++;
      continue;
      }
// ... ... 跳过类似的代码
// 如果有 /outdir 参数，则获取命令行中指定的目录路径
    if (args[i].Length > 8 && args[i].Substring(0,8).Equals("/outdir:"))
      {
      genpath = args[i].Substring(8);
      i++;
      continue;
      }
// 前面那么多的if相当于switch … case … default 块里面的 case 处理路径
// 下面这段代码即是 default 处理路径 – 如果命令行参数符合前面的if条件
// 都会执行里面的 continue 子句跳出循环，能执行到这里，说明参数
// 是无法识别的参数，因此报告错误并退出执行
    Abort("Unmatched switch = '"+args[i]+"'\nArguments are:\nmyc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
    }

  // 如果前面的循环执行完毕，还有参数列表未处理，说明输入了不支持的参数
  if (args.Length-i != 1)
    {
    Abort("myc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
}

  // 最后一个参数是要编译的源文件路径
  ifilename = args[args.Length-1]; // filename is last
  }
```

IO类中大部分函数都是为Tok.cs服务的，因此其它函数在解释词法分析的时候说明
