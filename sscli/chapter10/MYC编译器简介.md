在SSCLI里附带了两个示例编译器源码，用来演示CLR整个架构的弹性，一个是简化版的lisp编译器，一个是简化版的C编译器。lisp在国内用的少，因此这里我们主要看看C编译器的源码，源码位置是：`\sscli20\samples\compilers\myc`。

为了简单起见，该编译器实现了C语言的子集，如只支持 **int** 和 **void** 类型，可以声明静态和局部变量，但是局部变量只能在函数的顶部声明，只支持 **if-else**、**while** 和 **for** 等语句。编译器将C程序编译成MSIL语言，再调用IL编译器产生.NET程序，完整的语法如下表：

```
letter ::= "A-Za-z";
digit ::= "0-9";

name ::= letter { letter | digit };
integer ::= digit { digit };

ident ::= name | function_call;
function_call ::= name "(" [expr {, expr}] ")";

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

assign = ident "=" expr;
assign_stmt ::= assign ";" ;
if_stmt ::= "if" "(" expr ")" stmt_block [ "else" inner_block ];
while_stmt ::= "while" "(" expr ")" inner_block;
for_stmt ::= "for" "(" assign ";" expr ";" assign ")" inner_block
break_stmt ::= "break" ";";
cont_stmt ::= "continue" ";";
ret_stmt ::= "return" expr ";";
stmt ::= (
if_stmt
| while_stmt
| for_stmt
| break_stmt
| cont_stmt
| ret_stmt
| assign_stmt
);
inner_block ::= "{" { stmt } "}";
outer_block ::= "{" { inner_decl } { stmt } "}";

inner_decl ::= [ class ] type ident { "," ident } ";";

class ::= "extern" | "static" | "auto";
type ::= "int" | "void";

params ::= type ident { , type ident };
outer_decl ::= [ class ] type ident { "," ident } ";";
func_decl ::= [ class ] type ident "(" params ")" outer_block;
```

编译器支持的参数如下表：

**/debug**：生成程序时创建调试信息
**/nodebug**：不创建调试信息
**/list**	：输出MSIL中间源文件
**/dll**：创建一个DLL文件
**/exe**：创建一个可执行的.exe文件
**/outdir:path**：保存输出文件的文件夹

整个编译器的架构如下图所示：

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter10/myc-architecture.png)
