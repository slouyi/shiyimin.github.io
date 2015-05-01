.NET对象的内存布局
==================

每个虚拟机都有它自己的对象布局，本文我们将针对sscli源码和windbg调试器来查看不同类型的.net对象布局。

在.net虚拟机里，每个对象都需要保存这些信息：

*	对象的类型；
*	对象实例的成员属性（field）值；
*	hash值、锁信息等其他数据结构。

普通对象
-----------
在CLR里，对象在托管代码（managed code）和非托管代码（unmanaged code）里有不同的表现形式。在托管代码里，所有对象的基类Object类型是在 clr\src\bcl\system\Object.cs里定义，而其在非托管世界里，则复杂的多，由在clr\src\vm\object.h里定义的Object类型，clr\src\vm\SyncBlk.h里定义的ObjHeader等类型实现。对象在非托管代码理的内存物理布局如下图所示：

*	在非托管代码里，对象实际上有两个指针。
  *  object指针就是虚拟机返回给托管代码的对象引用地址，从这个指针开始，托管代码就可以获取到任何一个对象的类型以及成员变量信息。
 *  而另外一个指针objhead，实际上是非托管代码里，每个.NET对象实际的指针，在这个指针后面，包含了控制线程同步，甚至是COM Interop相关信息的SyncBlock索引，这个索引的作用我们会在后文提到。因为索引只用到32位字节，所以在64位系统运行的时候，会加上一个填充用的DWORD以便补齐内存边界。
* object指针后面紧跟的就是该对象所属的类型：MethodTable，MethodTable顾名思义就是函数表，在.NET里它就是对象类型的代表，在后文我们也会详细说明它。
*	类型信息后面就是每个对象实例成员变量的值信息了，如果是成员变量是引用类型，那么就保存被引用的对象的object指针（不是objhead），如果是值类型，比如说结构体，那么就按照值类型的内存布局，将变量值直接保存在对象的内存区域里。

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/object-memory-layout.jpg)
 
虽然.NET里所有引用类型的对象都从Object类型里继承，一些特殊的对象，在CLR里也在非托管代码里定义了一份不同的布局，方便虚拟机的处理。

数组对象 - Array
--------------------

在托管代码里，其在 clr\src\bcl\system\Array.cs 里定义，在非托管代码里，其在 clr\src\vm\object.h 里定义。之所以这样做，是因为数组对象即可以保存引用类型，也可以保存值类型，而且为了方便程序访问数组的长度，数组对象实际上是将长度信息直接保存在内存里了，如下图是其内存布局：

* 除了将长度信息直接保存到内存以外，如果是多维数组，则还会将各个纬度的上标和下标信息都保存到内存里，这主要是支持向VB这样的可以修改数组上下标的编程语言设计的。
* 如果是引用类型，则会把成员元素的类型指针保存在数组里；
* 如果是值类型，则直接保存成员元素的内容。 

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/array-memory-layout.jpg)
 
字符串对象
--------------
在托管代码里，其在 clr\src\bcl\system\String.cs 里定义，在非托管代码里，其在 clr\src\vm\object.h 里定义。

* 因为在.NET里字符串是不能修改的，可以将其当作数组处理，所以.NET直接将字符串的长度保存在内存里；
* 为了方便非托管代码处理字符串，每个字符串最后以 NULL 结尾，当然字符类型是WCHAR，而不是CHAR，这也就是说.NET下面字符默认是UNICODE。

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter2/string-memory-layout.jpg)
