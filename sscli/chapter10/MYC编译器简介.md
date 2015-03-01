��SSCLI�︽��������ʾ��������Դ�룬������ʾCLR�����ܹ��ĵ��ԣ�һ���Ǽ򻯰��lisp��������һ���Ǽ򻯰��C��������lisp�ڹ����õ��٣��������������Ҫ����C��������Դ�룬Դ��λ���ǣ�`\sscli20\samples\compilers\myc`��

Ϊ�˼�������ñ�����ʵ����C���Ե��Ӽ�����ֻ֧�� **int** �� **void** ���ͣ�����������̬�;ֲ����������Ǿֲ�����ֻ���ں����Ķ���������ֻ֧�� **if-else**��**while** �� **for** ����䡣��������C��������MSIL���ԣ��ٵ���IL����������.NET�����������﷨���±�

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

������֧�ֵĲ������±�

**/debug**�����ɳ���ʱ����������Ϣ
**/nodebug**��������������Ϣ
**/list**	�����MSIL�м�Դ�ļ�
**/dll**������һ��DLL�ļ�
**/exe**������һ����ִ�е�.exe�ļ�
**/outdir:path**����������ļ����ļ���

�����������ļܹ�����ͼ��ʾ��

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter10/myc-architecture.png)
