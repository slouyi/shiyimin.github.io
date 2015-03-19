语法分析
=======

MyC编译器采用自顶向下的方法进行语法解析，这种语法解析方式，一般是从最左边的Token开始，然后自顶向下看哪一条语法规则可能包含这个Token，如果包含这个Token，则自左向右根据这条语法规则逐一匹配后面的Token。自顶向下的语法解析我会在其他文章中说明，在前文我们已经列出了MyC的语法规则：

```
program ::= (
    outer_decl
  | func_decl
);

outer_decl ::= [ class ] type ident { "," ident } ";";
func_decl ::= [ class ] type ident "(" params ")" outer_block;
outer_block ::= "{" { inner_decl } { stmt } "}";
inner_decl ::= [ class ] type ident { "," ident } ";";
inner_block ::= "{" { stmt } "}";

ident ::= name | function_call;
function_call ::= name "(" [expr {, expr}] ")";
params ::= type ident { , type ident };

stmt ::= (
	if_stmt
	| while_stmt
	| for_stmt
	| break_stmt
	| cont_stmt
	| ret_stmt
	| assign_stmt
);

if_stmt ::= "if" "(" expr ")" stmt_block [ "else" inner_block ];
while_stmt ::= "while" "(" expr ")" inner_block;
for_stmt ::= "for" "(" assign ";" expr ";" assign ")" inner_block
break_stmt ::= "break" ";";
cont_stmt ::= "continue" ";";
ret_stmt ::= "return" expr ";";
assign_stmt ::= assign ";" ;

assign = ident "=" expr;
class ::= "extern" | "static" | "auto";
type ::= "int" | "void";

factor ::= (ident | integer | "(" expr ")" );
unary_factor ::= ["+"|"-"] factor;

term1 ::= ["*"|"/"] factor;
term0 ::= factor { term1 };
first_term ::= unary_factor term1;

math_expr ::= first_term { ["+"|"-"] term0 }
rel_expr ::= math_expr ("=="|"!="|"<"|">"|">="|"<=") math_expr;
not_factor ::= ["!"] rel_expr;
term_bool ::= not_factor { ("&" | "&&") not_factor };
bool_expr ::= term_bool { ("|" | "^") term_bool };
expr ::= bool_expr;

name ::= letter { letter | digit };
integer ::= digit { digit };

letter ::= "A-Za-z";
digit ::= "0-9";
```

我们用几个例子来说明自顶向下的解析过程，比如下面这个全局变量的定义：

```CSharp
int a;
```

采用自顶向下的过程如下：

*	首先编译器从最上面的语法规则 – program开始向下解析；
*	program包含两个规则，outer_decl和func_decl，因为outer_decl是第一个规则，所以编译器开始尝是当前的关键字是否匹配outer_decl的语法规则；

```
outer_decl ::= [ class ] type ident { "," ident } ";";
```

*	而组成outer_decl规则的class, type, ident都是语法规则，不是词法规则 – 即Token，这是因为在MyC的语法里，我们能找到class, type和ident的语法定义。其中 class 被方括号包围，说明其是可选的。
*	编译器向右再尝试看当前的token是否跟type匹配，发现第一个单词 int 跟type的语法规则匹配成功：

```
type ::= "int" | "void";
```

*	到这一步，编译器顺利消化掉第一个Token – int，再继续向右在outer_decl中匹配下一个Token，outer_decl规则里下一个要求的是ident。因此编译器判断下一个Token是不是匹配ident：

```
ident ::= name | function_call;
name ::= letter { letter | digit };
letter ::= "A-Za-z";
digit ::= "0-9";
```

*	此时词法分析器给出的下一个Token是a，匹配ident里的name规则（向下匹配路径是：ident -> name -> letter）。
*	编译器继续向右消化Token，此时源码中还有一个分号‘;’尚未匹配，而outer_decl里还有下面这些规则：

```
{ "," ident } ";";
```

*	然而outer_decl中，大括号里的规则是零到多个的可选项，因此源码里最后剩下的分号和outer_decl的最后一个分号匹配，编译器到这时顺利完成了一条C语句的分析，进而得知这条语句是一个全局变量的定义语句。
我们再举一个例子，来演示编译器是如何处理函数定义的：

```C
void main()
{
  int c;
  int d;
}
```

*	与前面分析全局变量的方式一样，编译器自顶向下解析，首先尝试outer_decl这个语法规则，并且从左向右根据 [class] type ident这些规则消化掉void main这两个Token，然而在处理括号‘(’编译器遇到麻烦，outer_decl里没有处理括号的规则。
*	编译器只好往左回溯，将已经处理掉的void和main两个token放回Token流，尝试program的第二个规则func_decl。注意：这里说的往左回溯，在人工编写的语法分析器里很少会这么做 – 因为这样的效率太低，在后面说明具体的源码的时候你会看到myc编译器是如何处理这种情况的。

```
func_decl ::= [ class ] type ident "(" params ")" outer_block;
```

*	与处理outer_decl的过程类似，编译器从左往右处理掉void和main两个Token，继续向右处理的时候，此时源码里的Token – ‘(’跟func_decl下一个规则 ”(” 是匹配的，因此编译器可以继续自顶向下，从左往右根据func_decl的规则消化源码里剩下的Token。
*	 在func_decl里有一个规则很有意思，inner_decl的定义和outer_decl的定义看起来是类似的，但是正是这样的区分使得编译器能够正确识别，同是 int a; 这样的语句，是全局变量定义还是函数内的局部变量定义 – 因为inner_decl只能从func_decl里递归解析到。
剩下的语句解析部分，我就不在这里多说了，请有兴趣的网友自己找几条C语句对着上面的语法自顶向下过一遍。
下面我们开始分析MyC的语法解析源码，这些功能都由parse类完成。Parse类的构造函数接受两个参数：Io对象和Tok对象，这里Io对象主要就是两个用处，一个是在语法解析过程中记录当前源码的位置，另一个是输出些错误消息，因此这里我们就不放什么篇幅说明Io对象了。

```CSharp
public Parse(Io i, Tok t)
{
  io = i;
  tok = t;
  // 初始化静态变量列表
  staticvar = new VarList();
}
```

Parse对象创建之后，实际的解析过程是由program函数处理的 – 即在myc.cs的Main函数里调用，大致浏览下源码，你应该就可以发现函数命名跟前面的语法规则名非常重合，如program、outerDecl等函数，因为语法解析的函数思路都差不多，这里我挑几个关键的函数说明下：

```CSharp
public void program()
{
  // 准备要生成的模块信息
  prolog();
  // 循环消耗源码里的Token流
  while (tok.NotEOF())
    {
  // 虽然名字叫outerDecl，实际上包含了两条语法规则，即program
  // 规则里的outer_decl和func_decl
    outerDecl();
    }
  // 错误处理
  if (Io.genexe && !mainseen)
    io.Abort("Generating executable with no main entrypoint");
  // 结束代码生成
  epilog();
}
```

MyC的语法很简单，因此在program函数里就一并将语法解析和代码生成做掉了。由于C语言是没有类的概念的，而.NET的IL却又是一个面向对象的中间语言，所以在program函数的一开始调用prolog函数，一方面是为正在编译的C程序生成一个默认的对象，另一方面，由于.NET的可执行文件Assembly实际上是可以由多个模块 – Module组成的，所以在prolog函数里也顺便生成了一个默认模块。下面是prolog函数的源码：

```CSharp
void prolog()
{
  // 创建代码生成对象
  emit = new Emit(io);
  // 准备最终包含C程序的.NET模块
  emit.BeginModule();		// need assembly module
  // 生成一个默认的类
  emit.BeginClass();
}
```

outerDecl函数里负责处理program的两个语法规则outer_decl和func_decl：

```CSharp
void outerDecl()
{
  // 保存当前解析的C语句中变量的信息，如变量名
  // 函数名、变量类型等信息
  Var e = new Var();
#if DEBUG
  Console.WriteLine("outerDecl token=["+tok+"]\n");
#endif
  // 记录当前源码位置，以便在结果IL文件（如果要生成IL的话）
  // 中保存位置信息
  CommentHolder();	/* mark the position in insn stream */

  // 处理 outer_decl 和 func_decl 规则共有的 [class] 规则
  dataClass(e);
  // 处理 outer_decl 和 func_decl 规则共有的 type 规则
  dataType(e);

  // 判断下一个字符是否是左括号，如果是的话，则按
  // func_decl规则处理
  if (io.getNextChar() == '(')
    declFunc(e);
  // 否则按outer_decl规则处理
  else
    declOuter(e);
}

// 解析outer_decl语法规则的剩余部分
void declOuter(Var e)
  {
#if DEBUG
  Console.WriteLine("declOuter1 token=["+tok+"]\n");
#endif
  // 前面在outerDecl函数里已经处理过 [class] 和 type 规则了
  // 因此目前Token流的第一个Token时ident，也就是变量名
  // 这里将变量名赋值给e - 即由outerDecl创建的变量名
  e.setName(tok.getValue());	/* use value as the variable name */
  // 将这个变量保存到全局变量列表里，以备后面语义分析时使用
  addOuter(e);		/* add this variable */
  // 在结果语法树里创建一个变量声明节点
  emit.FieldDef(e);		/* issue the declaration */
  // 如果当前语句有匹配 [class] 规则的部分，即语句的前面有static, 
  // extern 这些关键字，把这个信息也保存到变量声明节点里，以便 
  // 后面生成代码时参考
  if (e.getClassId() == Tok.T_DEFCLASS)
    e.setClassId(Tok.T_STATIC);	/* make sure it knows its storage class */

  // 处理 outer_decl 规则里 { "," ident } 这个多变量声明部分
  // 即处理类似：int a, b, c; 这样的变量声明语句
  // 这个过程是通过判断后面的Token是否是 ',' 来完成的
  /*
   * loop while there are additional variable names
   */
  while (io.getNextChar() == ',')
    {
    // 后面跟着 ','，那么先消化掉这个字符
    tok.scan();
    if (tok.getFirstChar() != ',')
	io.Abort("Expected ','");
    // 向右扫描
    tok.scan();
    // 尝试找到一个匹配 ident 规则的Token
    if (tok.getId() != Tok.T_IDENT)
	io.Abort("Expected identifier");
    // 找到一个变量名 - 即匹配 ident 规则的Token
    e.setName(tok.getValue()); /* use value as the variable name */
    // 将这个新的全局变量添加到全局变量表里
    addOuter(e);		/* add this variable */
    // 当然也要在结果语法树里保存这个变量声明节点
    emit.FieldDef(e);		/* issue the declaration */
    if (e.getClassId() == Tok.T_DEFCLASS)
      e.setClassId(Tok.T_STATIC);	/* make sure it knows its storage class */
    }

  // 消化完前面的变量定义，看看后面跟着的字符是不是分号 - ';'
  /*
   * move beyond end of statement indicator
   */
  tok.scan();
  if (tok.getFirstChar() != ';')
    io.Abort("Expected ';'");
  // 顺利解析完一条语句，扫尾处理
  CommentFill();
  tok.scan();
#if DEBUG
  Console.WriteLine("declOuter2 token=["+tok+"]\n");
#endif
  }
```
