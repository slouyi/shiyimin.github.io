从源码解读.NET对象内存布局
==========================

前面我们图解了.NET里各种对象的内存布局，我们再来从调试器和clr源码的角度来看一下对象的内存布局。我写了一个测试程序来加深对.net对象内存布局的了解：
```
using System;
using System.Runtime.InteropServices;

// 实际上是一个C语言里的联合体
[StructLayout(LayoutKind.Explicit)]
public struct InnerStruct
{
    [FieldOffset(0)]
    public float FloatValue;

    [FieldOffset(0)]
    public double DoubleValue;
}

public struct TestStruct
{
    public int IntValue;

    public string StringValue;

    public object ObjectValue;

    public InnerStruct InnerStructValue;
}

public class ObjectLayout
{
    private int _IntValue = 456;
    public int IntValue
    {
        get { return _IntValue; }
        set { _IntValue = value; }
    }

    public static void Main()
    {
        Object o = new Object();

        lock (o)
        {
            Console.WriteLine("Object实例: {0}", o.ToString());
        }

        int i = 123;
        Console.WriteLine("int值: {0}", i);

        string s = "This is a string";
        Console.WriteLine("字符串：{0}", s);

        ObjectLayout[] olArr = new ObjectLayout[10];
        olArr[0] = new ObjectLayout();
        olArr[0].IntValue = 2222;
        Console.WriteLine("数组的长度：{0}", olArr.Length);

        object[] objArr = new object[2];
        objArr[0] = o;
        Console.WriteLine("数组的长度：{0}", objArr.Length);

        string[] strArr = new string[2];
        strArr[0] = s;
        strArr[1] = s + "!";
        Console.WriteLine("数组的长度：{0}", strArr.Length);

        TestStruct ts = new TestStruct();
        ts.IntValue = 100;
        ts.StringValue = s + "!";
        ts.ObjectValue = o;
        ts.InnerStructValue.FloatValue = 789.0f;

        int[] intArr = new int[10];
        for (int j = 0; j < intArr.Length; ++j)
        {
            intArr[j] = j;
        }
        Console.WriteLine("int数组的长度：{0}", intArr.Length);

        TestStruct[] tsArr = new TestStruct[2];
        tsArr[0] = ts;
        Console.WriteLine("TestStruct数组的长度：{0}", tsArr.Length);
    }
}
```

使用命令编译一个调试版本的objectlayout.exe程序：`csc /debug objectlayout.cs`

用sos浏览对象内存布局
---------------------

我们用sos这个工具加深对.net对象的理解，sos可以在Visual Studio里使用：

* 启动VS，依次点击菜单里的“文件（File）” -> “打开（Open）” -> “工程或解决方案（Project/Solution）”，然后选择刚刚编译的objectlayout.exe程序，开始调试这个程序；
* 对于托管程序，VS支持多种调试模式，如果要使用sos插件的话，需要采用“混合（Mixed）”调试模式调试程序，具体做法是在“解决方案管理器（Solution Explorer）”里右键单击objectlayout.exe程序，然后点击“属性（Properties）”打开属性窗口，将里面的“调试器类型（Debugger Type）”改成“混合（Mixed）”模式；

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/objectlayout-debug-property-page.png)

* 在VS里打开程序的源码objectlayout.cs，并在Main函数的最后一行设置断点；
在VS里，打开“立即”窗口，菜单命令是：“调试（Debug）” -> “窗口（Windows）” -> “立即（Immediate）”；
* 在“立即”窗口里，执行命令将sos加载到VS中：`!load C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\sos.dll`

这个时候就可以在vs里使用sos插件里面的命令了，如下图所示：

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/load-sos-in-vs.png)

这里对sos命令不做过多的解释，有兴趣的网友可以参看我的[《Windows调试技术》](http://product.china-pub.com/3502685) 视频来了解sos的用法，下面我用类似bash注释的方式解释查看过程：

```
＃
＃使用 !clrstack 命令查看当前被调试进程的堆栈，而 -l 参数则告诉sos同时
＃ 显示堆栈上每个函数的局部变量。
＃
!clrstack -l
OS Thread Id: 0xba4 (2980)
ESP       EIP     
0012f408 00e10341 ObjectLayout.Main()

＃
＃ 下面打印了Main函数里的所有局部变量，如果执行过GC，有些变量可能不可见
＃ 局部变量左边是局部变量的内存地址，而右边则是局部变量的值，因为大部分
＃ 局部变量都是引用类型，所有值大部分都是指针，除了少数几个，如第二个变量
＃ 就是一个值类型，因此直接保存了它的值：0x0000007b
＃
    LOCALS:
        0x0012f444 = 0x012b1bd8
        0x0012f440 = 0x0000007b
        <CLR reg> = 0x012b1af4
        0x0012f438 = 0x012c9520
        0x0012f434 = 0x012c966c
        0x0012f430 = 0x012c9788
        0x0012f41c = 0x012c98d8
        0x0012f418 = 0x012c990c
        0x0012f414 = 0x0000000a
        0x0012f410 = 0x012c9a5c
        0x0012f40c = 0x00000000

0012f69c 79e88f63 [GCFrame: 0012f69c] 
```

接下来，我们一个个分析这些对象的内存布局，首先是第一个对象 － object类型的o：

```
!do 0x012b1bd8
Name: System.Object               ＃指明了类型，这个类型由保存在对象里的MethodTable获取
MethodTable: 790f9c18             ＃ MethodTable地址，直接保存在对象里
EEClass: 790f9bb4                   ＃通过MethodTable解析到
Size: 12(0xc) bytes
 (C:\WINDOWS\assembly\GAC_32\mscorlib\2.0.0.0__b77a5c561934e089\mscorlib.dll)
Object
Fields:
None
```

打开VS的“内存（Memory）”窗口，或者“命令（Command）”窗口，查看0x012b1bd8地址处的内存，这里为了写文章方便，我用的是“命令”窗口，在VS里依次点击菜单“视图（View）” -> “其他窗口（Other Windows）” -> “命令窗口（Command Window）”。在命令窗口里执行下面的命令（熟悉windbg的同学应该知道这是windbg里的命令）：

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/vs-command-window.png)


```
＃
＃ 你可以直接给出变量名，vs会自动将变量名解析为内存地址
＃ 可以看到，对象的第一个指针就是说明自己类型的MethodTable
＃ 指针 －> 790f9c18，
＃
>dd o
0x012B1BD8  790f9c18 00000000 00000000 00000000 
#
# 当然也可以直接给dd命令内存地址
#
>dd 0x012b1bd8
0x012B1BD8  790f9c18 00000000 00000000 00000000 
```

前文我们已经提到clr将对象的指针做了一些处理，对托管代码隐藏了objheader信息，这个信息其中一个作用就是处理线程同步信息，要看看syncvalue是怎么工作的话，可以重新启动被调试程序，并将程序中断在代码的第39行即lock语句那里 － 在其执行之前中断程序，如下图所示：

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/set-bp-on-lock-statement.png)

然后我们在“命令窗口”查看对象的内存布局：

```
＃
＃我用的是虚拟机上安装的32位xp系统，因此我们将地址提前
＃一个指针，看看当前objheader的synvalue的值，目前因为
＃没有线程需要同步访问这个对象，所以其值为0
＃
>dd 0x012b1bd4
0x012B1BD4  00000000 790f9c18 00000000 00000000 
 
＃
＃单步执行lock语句一次，以便开始线程同步，再看相同地址的值
＃现在会发现lockvalue已经更新了，lockvalue的作用在后文说明
＃这里就不详细说明它了。
＃
>dd 0x012b1bd4
0x012B1BD4  00000001 790f9c18 00000000 00000000 
```

我们再看第三个局部变量，string类型的s，使用sos命令查看的结果如下：

```
!do 0x012b1af4
Name: System.String
MethodTable: 790fa3e0
EEClass: 790fa340
Size: 50(0x32) bytes
 (C:\WINDOWS\assembly\GAC_32\mscorlib\2.0.0.0__b77a5c561934e089\mscorlib.dll)
＃
＃对于字符串对象，!do命令足够聪明，可以直接将字符串的内容打印出来
＃
String: This is a string
＃
＃显示该对象实例的每一个成员变量的值
＃
Fields:
      MT    Field   Offset                 Type VT     Attr    Value Name
790fed1c  4000096        4         System.Int32  0 instance       17 m_arrayLength
790fed1c  4000097        8         System.Int32  0 instance       16 m_stringLength
790fbefc  4000098        c          System.Char  0 instance       54 m_firstChar
790fa3e0  4000099       10        System.String  0   shared   static Empty
    >> Domain:Value  0014c558:790d6584 <<
79124670  400009a       14        System.Char[]  0   shared   static WhitespaceChars
    >> Domain:Value  0014c558:012b1670 <<
```

我们再在命令行里用dd查看它的内存布局，因为是字符串，所以这里我们用dc命令，除了用16进制显示内存以外，还尽量使用字符的形式打印内存的每个指针：

```
＃
＃第一个指针保存的仍然是对象的类型信息 － MethodTable指针，
＃接下来第二个指针就是如果将字符串当作数组看待的话，它的长度，
＃这个长度会包括最后的’\0’，这里它的值是 0x11，也就是17。
＃第三个指针就是字符串的长度，即 0x10，也就是16个字符。
＃在字符串长度后面就是实际的字符串WCHAR数组了。
＃
>dc s
0x012B1AE8  790fa3e0 00000011 00000010 00680054  à£.y........T.h.
0x012B1AF8  00730069 00690020 00200073 00200061  i.s. .i.s. .a. .
0x012B1B08  00740073 00690072 0067006e 00000000  s.t.r.i.n.g.....
0x012B1B18  00000000 790fa3e0 00000008 00000007  ....à£.y........
```

接下来我们再来看引用类型的数组对象在内存里的布局，下面是 ObjectLayout[] 类型的对象olArr的结果，可以看到在clr里，对象的类型实际上 System.Object[]，而不是 ObjectLayout[]。

```
!do 0x012c9520
Name: System.Object[]
＃
＃MethodTable的值是 79124228，跟后面 object[] 类型对象的 objArr 的
＃MethodTable是一样的
＃
MethodTable: 79124228
EEClass: 7912479c
Size: 56(0x38) bytes
＃
＃对于数组对象，sos会打印出数组的维度和大小信心
＃
Array: Rank 1, Number of elements 10, Type CLASS
＃
＃数组元素的类型
＃
Element Type: ObjectLayout
Fields:
None

＃
＃使用 da 命令可以打印出数组的详细内容
＃
!da 0x012c9520
Name: ObjectLayout[]
MethodTable: 79124228
EEClass: 7912479c
Size: 56(0x38) bytes
Array: Rank 1, Number of elements 10, Type CLASS
Element Methodtable: 00933018
[0] 012c9558
[1] null
[2] null
[3] null
[4] null
[5] null
[6] null
[7] null
[8] null
[9] null
```

把olArr对象的内存打印出来，可以看到：

* 第一个指针跟其他对象一样，是MethodTable，也就是对象类型的指针；
* 第二个指针是数组的大小0xa，也就是10；
* 第三个指针是数组里元素的类型指针；
* 第四个指针开始则是各个对象的引用。

```
>dd olArr
0x012C9438  79124228 0000000a 00933018 012c9558  
0x012C9448  00000000 00000000 00000000 00000000 

＃
＃打印 object[] 类型的 objArr 对象的信息
＃
!do 0x012c966c
Name: System.Object[]
＃
＃MethodTable指针与前面的ObjectLayout[]对象的MethodTable完全一样
＃
MethodTable: 79124228
EEClass: 7912479c
Size: 24(0x18) bytes
Array: Rank 1, Number of elements 2, Type CLASS
Element Type: System.Object
Fields:
None

!da 0x012c966c
Name: System.Object[]
MethodTable: 79124228
EEClass: 7912479c
Size: 24(0x18) bytes
Array: Rank 1, Number of elements 2, Type CLASS
Element Methodtable: 790f9c18
[0] 012b1c3c
[1] null

＃
＃打印 string[] 类型的 strArr 对象的信息
＃
!do 0x012c9788
Name: System.Object[]

＃
＃MethodTable指针与前面的ObjectLayout[]和object[]对象的MethodTable完全一样
＃
MethodTable: 79124228
EEClass: 7912479c
Size: 24(0x18) bytes
Array: Rank 1, Number of elements 2, Type CLASS
Element Type: System.String
Fields:
None

!da 0x012c9788
Name: System.String[]
MethodTable: 79124228
EEClass: 7912479c
Size: 24(0x18) bytes
Array: Rank 1, Number of elements 2, Type CLASS
Element Methodtable: 790fa3e0
[0] 012b1af4
[1] 012c97a0

＃
＃打印 int[] 类型的 intArr 对象的信息
＃
!da 0x012c990c
Name: System.Int32[]
＃
＃注意：MethodTable 也就是类型指针跟前面引用类型的MethodTable不同
＃
MethodTable: 791240f0
EEClass: 791241a8
Size: 52(0x34) bytes
Array: Rank 1, Number of elements 10, Type Int32
Element Methodtable: 790fed1c
[0] null
[1] 00000001
[2] 00000002
[3] 00000003
[4] 00000004
[5] 00000005
[6] 00000006
[7] 00000007
[8] 00000008
[9] 00000009
```

查看intArr的内存布局，可以看到，与前面的引用类型数组不同，第三个指针就是数组的第一个元素（值为0）了，而引用类型数组的第三个指针是元素的类型指针。
```
>dd intArr
0x012C97B0  791240f0 0000000a 00000000 00000001  
0x012C97C0  00000002 00000003 00000004 00000005  
0x012C97D0  00000006 00000007 00000008 00000009

＃
＃打印自定义结构体 TestStruct[] 类型的 tsArr对象的信息
＃
!da 0x012c9a5c
Name: TestStruct[]
＃
＃注意：MethodTable 指针不仅跟前面引用类型的MethodTable不同，
＃而且跟intArr的类型也不一样
＃
MethodTable: 00933200
EEClass: 00933180
Size: 52(0x34) bytes
Array: Rank 1, Number of elements 2, Type VALUETYPE
Element Methodtable: 0093313c
[0] 012c9a64
[1] 012c9a78
```

查看tsArr的内存布局，也可以看到，clr直接将结构体的所有内容保存在数组元素的内存空间里，与 int[] 类型的对象一样，结构体数组也不保存元素的类型信息。

```
>dd tsArr
0x012C98B8  00933200 00000002 012c977c 012b1bd8  
0x012C98C8  00000064 44454000 00000000 00000000  
0x012C98D8  00000000 00000000 00000000 00000000  
0x012C98E8  00000000 00000000 00000000 00000000  
＃
＃使用df命令，打印出内存里的浮点数，可以看到在结构体的最后一个指针
＃就是浮点数的值
＃
>df tsArr
0x012C98B8   1.3517755e-038  2.803e-045#DEN  3.1700095e-038  3.1427717e-038  
0x012C98C8   1.401e-043#DEN       789.00000      0.00000000      0.00000000  
0x012C98D8       0.00000000      0.00000000      0.00000000      0.00000000  
0x012C98E8       0.00000000      0.00000000      0.00000000      0.00000000 
＃
＃使用dc命令查看数组第一个元素的第一个指针，是结构体的字符串成员变量
＃通过这个例子也可以看到，在实际的内存布局里，如果不显示指定成员变量的
＃内存布局，clr里对象的成员变量的布局顺序跟源码的顺序有可能是不一样的
＃
>dc 0x012c977c
0x012C977C  790fa3e0 00000012 00000011 00680054  à£.y........T.h.
0x012C978C  00730069 00690020 00200073 00200061  i.s. .i.s. .a. .
0x012C979C  00740073 00690072 0067006e 00000021  s.t.r.i.n.g.!...
0x012C97AC  00000000 791240f0 0000000a 00000000  ….ð@.y........
```

通过前面的分析，可以看到，实际上所有的引用类型数组的类型都是一样的，即 object[] 类型，但是值类型数组的类型却各不相同，这个差异在jit的时候就已经决定了，也就是说，虽然在 IL 代码里，创建数组的指令都是 newarr 指令，但是在jit编译生成代码后，传给 newarr 指令的类型参数就已经不一样了。我们在后面解读 jit 源码的时候会继续提到这一点。
