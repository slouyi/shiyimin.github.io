前面讲过语法的解析之后，代码生成方面就简单很多了。虽然myc是一个简单的示例编译器，但是它还是在解析的过程中生成了一个小的语法树，这个语法树将会用在生成exe可执行文件和il源码的过程中。

编译器在解析时，使用emit类来产生中间的语法树，语法树的数据结构和操作方法在iasm这个类型里完成，源程序的语法解析完毕后，Exe和Asm两个类分别遍历生成的语法树产生最终的代码。

我们来看几个代码的例子，下表的函数 Parser.program 里，在函数开始和结束的地方分别调用了 prolog 和 epilog 两个函数，这两个函数的目的就是在语法解析的前后执行一些准备和扫尾工作。如在编译过程的开始阶段，根据.net assembly的要求创建好模块（module）和类（class），虽然c语言是一个面向过程的语言，但是在.net是一个面向对象的环境，所有的代码都应该保存在一个类里。

```
public void program()
  {
  prolog();
  while (tok.NotEOF())
    {
    outerDecl();
    }
  if (Io.genexe && !mainseen)
    io.Abort("Generating executable with no main entrypoint");
  epilog();
  }

void prolog()
  {
  emit = new Emit(io);
  emit.BeginModule();    	// need assembly module
  emit.BeginClass();
  }
```

而在emit里，BeginModule和BeginClass这两个函数的代码如下：

```
public void BeginModule()
  {
  // 委托给Exe类来创建这个module，虽然在.net里，一个assembly可以由
  // 多个module组成，但是在C程序里，只要一个module就足够了，因此
  // 下面的代码并没有在生成IL源码的时候产生module。
  exe.BeginModule(io.GetInputFilename());
  }

public void BeginClass()
  {
  // 委托给Exe类来创建class，再进一步跟踪代码的时候，会发现它其实
  // 是根据反射技术来创建类型的。
  exe.BeginClass(Io.GetClassname(), TypeAttributes.Public);
  // 如果在执行程序的命令行里，启用了生成源码的开关，那么
  // 将会输出IL class的源码定义。
  if (Io.genlist)
    io.Out(".class " + Io.GetClassname() + "{\r\n");
  }
```

.NET里，可以使用反射技术来生成assembly、类型和函数，下表就是Exe类的BeginModule函数的源码：

```
public void BeginModule(string ifile)
  {
  // .net的动态assembly创建功能，要求跟appdomain绑定
  appdomain = System.Threading.Thread.GetDomain();
  appname = getAssemblyName(filename);
  // 调用AppDomain.DefineDynamiceAssembly创建一个Assembly，以这个为
  // 起点，可以创建类型，创建函数并执行。实际上，.net上的IronPython等
  // 动态语言的实现就非常依赖这个技术。
  appbuild = appdomain.DefineDynamicAssembly(appname,
					     AssemblyBuilderAccess.Save,
					     Io.genpath);
  // 在.net里，所有的代码实际上都应该保存在一个module里。
  emodule = appbuild.DefineDynamicModule(
  		filename+"_module",
  		Io.GetOutputFilename(),
		Io.gendebug);
  Guid g = System.Guid.Empty;
  if (Io.gendebug)
    srcdoc = emodule.DefineDocument(ifile, g, g, g);
  }
```

准备工作做好了以后，就可以生成语法树了，编译器在解析语法的过程当中，不停的往语法树里添加元素，如在编译函数的过程中，以处理while循环为例（其中一个调用路径是：`program -> outerDecl -> declFunc -> blockOuter -> fcWhile`）

```
void fcWhile()
  {
  // 在一般的il或者汇编语言里，循环和判断语句一般都是在不同路径的入口
  // 出定义好标签（label），再通过判断条件的方式跳转到指定的label实现的
  String label1 = newLabel();
  String label2 = newLabel();
 
  // 记录当前源码的位置，以便生成IL源码的时候可以把源代码和IL代码对照生成
  CommentHolder();	/* mark the position in insn stream */
  // 一般来说，循环语句至少有两个分支代码块，一个是继续循环的代码块，
  // 一个是跳出循环的代码块，看后面的代码，这个label是循环中执行的代码块
  // 开始的地方，以便满足条件的时候跳到开头继续执行
  emit.Label(label1);
  tok.scan();
  // 做一些错误判断
  if (tok.getFirstChar() != '(')
    io.Abort("Expected '('");

  // 处理循环条件的判断语句相关代码
  boolExpr();
  CommentFillPreTok();
  // 跳出循环的label
  emit.Branch("brfalse", label2);

  // 循环内部的代码块，进入blockInner进行循环里面的编译
  blockInner(label2, label1);	/* outer label, top of loop */
  // 如果满足循环条件，跳转到代码块开头继续执行
  emit.Branch("br", label1);
  
  // 循环结束，跳出循环的地方
  emit.Label(label2);
  }
```

而在emit类型里，各个方法只是将解析出来的语法元素添加到语法树里，语法树的节点、数据结构和操作方法都在IAsm这个类里定义，如下表是 Branch 的源码：

```
public void Branch(String s, String lname)
  {				// this is the branch source
  NextInsn(1);
  // 往语法树里添加一个类别为 Branch 的元素
  icur.setIType(IAsm.I_BRANCH);
  // 指令名称
  icur.setInsn(s);
  // 指令参数
  icur.setLabel(lname);
  }
```

当程序编译完成后，Exe类和Asm类则分别遍历语法树生成最终的结果，在myc编译器的源码里，Parser.declFunc函数通过调用Emit.IL函数来完成程序的生成：

```
// 因为C程序大部分都是由函数组成的，而且函数使用到的变量或者其他函数，
// 都必须在函数之前定义，所以只需要在解析函数的时候实时生成代码即可
void declFunc(Var e)
  {
#if DEBUG
  Console.WriteLine("declFunc token=["+tok+"]\n");
#endif
  CommentHolder();	// start new comment
  // 记录解析出来的函数名
  e.setName(tok.getValue()); /* value is the function name */
  // 如果函数名是main，则设置一个标识位 - mainseen为true
  // 在外层的函数里，会通过判断这个标志来确定程序是否有语义错误
  if (e.getName().Equals("main"))
    {
    if (Io.gendll)
      io.Abort("Using main entrypoint when generating a DLL");
    mainseen = true;
    }
  // 函数名也是一个全局变量，放到全局变量表里，以便做语义分析
  // 例如要调用的函数之前没有定义，则应该报错，在后文我们将
  // 看到语义方面的处理
  staticvar.add(e);		/* add function name to static VarList */
  paramvar = paramList();	// track current param list
  e.setParams(paramvar);	// and set it in func var
  // 记录函数里面定义的局部变量
  localvar = new VarList();	// track new local parameters
  CommentFillPreTok();

  // 开始生成函数的prolog，例如参数传递，this对象等
  emit.FuncBegin(e);
  if (tok.getFirstChar() != '{')
    io.Abort("Expected ‘{'");
  // 递归分析函数里面的源码
  blockOuter(null, null);
  emit.FuncEnd();
  
  // 解析完整个函数后，执行代码生成操作
  emit.IL();
  // 如果需要生成IL源码，则调用LIST函数生成IL源码
  if (Io.genlist)
    emit.LIST();
  emit.Finish();
  }
```

而emit.IL函数就是用Exe类型遍历整个语法树，生成结果程序：

```
public void IL()
  {
  IAsm a = iroot;
  IAsm p;

  // 循环遍历整个语法树
  while (a != null)
    {
    // 根据语法树里各个节点的类型来执行对应的操作
    switch (a.getIType())
      {
      case IAsm.I_INSN:
	exe.Insn(a);
	break;
      case IAsm.I_LABEL:
	exe.Label(a);
	break;
      case IAsm.I_BRANCH:
	exe.Branch(a);
	break;
      // 省略一些代码
      default:
	io.Abort("Unhandled instruction type " + a.getIType());
	break;
      }
    p = a;
    a = a.getNext();
    }
  }
```

而Exe类型执行真正的代码生成，如前面IL函数，在碰到I_BRANCH类型的节点时，调用Exe.Branch函数在动态Assembly (DynamicAssemby) 里生成代码：

```
public void Branch(IAsm a)
  {
  Object o = opcodehash[a.getInsn()];
  if (o == null)
    Io.ICE("Instruction branch opcode (" + a.getInsn() + ") not found in hash”);
  // 使用 ILGenerator 类生成跳转IL指令。
  il.Emit((OpCode) o, (Label) getILLabel(a));
  }
```

而Asm类也采用类似的方法生成IL源码。

最后，myc编译器里也有一些语义方面的处理，如前面讲到的函数调用时，如果被调用的函数没有定义的话，应该抛出异常的情况，在Parser.statement（即编译实际的C语句的函数）中就有所体现

```
void statement()
  {
  Var e;
  String vname = tok.getValue();

  CommentHolder();	/* mark the position in insn stream */
  switch (io.getNextChar())
    {
    case '(':			/* this is a function call */
      // 省略一些语法处理方面的代码
     tok.scan();		/* move to next token */
     // 下面这一行即在生成函数调用代码之前，在全局变量列表里
   // 查找要调用的函数是否已经定义了，如果没有定义，则应该报告此错误
      e = staticvar.FindByName(vname); /* find the symbol (e cannot be null) */
      emit.Call(e);
     
      // 省略后面的代码
   }
  if (tok.getFirstChar() != ';')
    io.Abort("Expected ';'");
  tok.scan();
  }
```
