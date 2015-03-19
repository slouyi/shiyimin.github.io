�﷨����
=======

MyC�����������Զ����µķ��������﷨�����������﷨������ʽ��һ���Ǵ�����ߵ�Token��ʼ��Ȼ���Զ����¿���һ���﷨������ܰ������Token������������Token�����������Ҹ��������﷨������һƥ������Token���Զ����µ��﷨�����һ�������������˵������ǰ�������Ѿ��г���MyC���﷨����

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

�����ü���������˵���Զ����µĽ������̣������������ȫ�ֱ����Ķ��壺

```CSharp
int a;
```

�����Զ����µĹ������£�

*	���ȱ���������������﷨���� �C program��ʼ���½�����
*	program������������outer_decl��func_decl����Ϊouter_decl�ǵ�һ���������Ա�������ʼ���ǵ�ǰ�Ĺؼ����Ƿ�ƥ��outer_decl���﷨����

```
outer_decl ::= [ class ] type ident { "," ident } ";";
```

*	�����outer_decl�����class, type, ident�����﷨���򣬲��Ǵʷ����� �C ��Token��������Ϊ��MyC���﷨��������ҵ�class, type��ident���﷨���塣���� class �������Ű�Χ��˵�����ǿ�ѡ�ġ�
*	�����������ٳ��Կ���ǰ��token�Ƿ��typeƥ�䣬���ֵ�һ������ int ��type���﷨����ƥ��ɹ���

```
type ::= "int" | "void";
```

*	����һ����������˳����������һ��Token �C int���ټ���������outer_decl��ƥ����һ��Token��outer_decl��������һ��Ҫ�����ident����˱������ж���һ��Token�ǲ���ƥ��ident��

```
ident ::= name | function_call;
name ::= letter { letter | digit };
letter ::= "A-Za-z";
digit ::= "0-9";
```

*	��ʱ�ʷ���������������һ��Token��a��ƥ��ident���name��������ƥ��·���ǣ�ident -> name -> letter����
*	������������������Token����ʱԴ���л���һ���ֺš�;����δƥ�䣬��outer_decl�ﻹ��������Щ����

```
{ "," ident } ";";
```

*	Ȼ��outer_decl�У���������Ĺ������㵽����Ŀ�ѡ����Դ�������ʣ�µķֺź�outer_decl�����һ���ֺ�ƥ�䣬����������ʱ˳�������һ��C���ķ�����������֪���������һ��ȫ�ֱ����Ķ�����䡣
�����پ�һ�����ӣ�����ʾ����������δ���������ģ�

```C
void main()
{
  int c;
  int d;
}
```

*	��ǰ�����ȫ�ֱ����ķ�ʽһ�����������Զ����½��������ȳ���outer_decl����﷨���򣬲��Ҵ������Ҹ��� [class] type ident��Щ����������void main������Token��Ȼ���ڴ������š�(�������������鷳��outer_decl��û�д������ŵĹ���
*	������ֻ��������ݣ����Ѿ��������void��main����token�Ż�Token��������program�ĵڶ�������func_decl��ע�⣺����˵��������ݣ����˹���д���﷨����������ٻ���ô�� �C ��Ϊ������Ч��̫�ͣ��ں���˵�������Դ���ʱ����ῴ��myc����������δ�����������ġ�

```
func_decl ::= [ class ] type ident "(" params ")" outer_block;
```

*	�봦��outer_decl�Ĺ������ƣ��������������Ҵ����void��main����Token���������Ҵ����ʱ�򣬴�ʱԴ�����Token �C ��(����func_decl��һ������ ��(�� ��ƥ��ģ���˱��������Լ����Զ����£��������Ҹ���func_decl�Ĺ�������Դ����ʣ�µ�Token��
*	 ��func_decl����һ�����������˼��inner_decl�Ķ����outer_decl�Ķ��忴���������Ƶģ�������������������ʹ�ñ������ܹ���ȷʶ��ͬ�� int a; ��������䣬��ȫ�ֱ������廹�Ǻ����ڵľֲ��������� �C ��Ϊinner_declֻ�ܴ�func_decl��ݹ��������
ʣ�µ����������֣��ҾͲ��������˵�ˣ�������Ȥ�������Լ��Ҽ���C������������﷨�Զ����¹�һ�顣
�������ǿ�ʼ����MyC���﷨����Դ�룬��Щ���ܶ���parse����ɡ�Parse��Ĺ��캯����������������Io�����Tok��������Io������Ҫ���������ô���һ�������﷨���������м�¼��ǰԴ���λ�ã���һ�������Щ������Ϣ������������ǾͲ���ʲôƪ��˵��Io�����ˡ�

```CSharp
public Parse(Io i, Tok t)
{
  io = i;
  tok = t;
  // ��ʼ����̬�����б�
  staticvar = new VarList();
}
```

Parse���󴴽�֮��ʵ�ʵĽ�����������program��������� �C ����myc.cs��Main��������ã����������Դ�룬��Ӧ�þͿ��Է��ֺ���������ǰ����﷨�������ǳ��غϣ���program��outerDecl�Ⱥ�������Ϊ�﷨�����ĺ���˼·����࣬�������������ؼ��ĺ���˵���£�

```CSharp
public void program()
{
  // ׼��Ҫ���ɵ�ģ����Ϣ
  prolog();
  // ѭ������Դ�����Token��
  while (tok.NotEOF())
    {
  // ��Ȼ���ֽ�outerDecl��ʵ���ϰ����������﷨���򣬼�program
  // �������outer_decl��func_decl
    outerDecl();
    }
  // ������
  if (Io.genexe && !mainseen)
    io.Abort("Generating executable with no main entrypoint");
  // ������������
  epilog();
}
```

MyC���﷨�ܼ򵥣������program�������һ�����﷨�����ʹ������������ˡ�����C������û����ĸ���ģ���.NET��ILȴ����һ�����������м����ԣ�������program������һ��ʼ����prolog������һ������Ϊ���ڱ����C��������һ��Ĭ�ϵĶ�����һ���棬����.NET�Ŀ�ִ���ļ�Assemblyʵ�����ǿ����ɶ��ģ�� �C Module��ɵģ�������prolog������Ҳ˳��������һ��Ĭ��ģ�顣������prolog������Դ�룺

```CSharp
void prolog()
{
  // �����������ɶ���
  emit = new Emit(io);
  // ׼�����հ���C�����.NETģ��
  emit.BeginModule();		// need assembly module
  // ����һ��Ĭ�ϵ���
  emit.BeginClass();
}
```

outerDecl�����︺����program�������﷨����outer_decl��func_decl��

```CSharp
void outerDecl()
{
  // ���浱ǰ������C����б�������Ϣ���������
  // ���������������͵���Ϣ
  Var e = new Var();
#if DEBUG
  Console.WriteLine("outerDecl token=["+tok+"]\n");
#endif
  // ��¼��ǰԴ��λ�ã��Ա��ڽ��IL�ļ������Ҫ����IL�Ļ���
  // �б���λ����Ϣ
  CommentHolder();	/* mark the position in insn stream */

  // ���� outer_decl �� func_decl �����е� [class] ����
  dataClass(e);
  // ���� outer_decl �� func_decl �����е� type ����
  dataType(e);

  // �ж���һ���ַ��Ƿ��������ţ�����ǵĻ�����
  // func_decl������
  if (io.getNextChar() == '(')
    declFunc(e);
  // ����outer_decl������
  else
    declOuter(e);
}

// ����outer_decl�﷨�����ʣ�ಿ��
void declOuter(Var e)
  {
#if DEBUG
  Console.WriteLine("declOuter1 token=["+tok+"]\n");
#endif
  // ǰ����outerDecl�������Ѿ������ [class] �� type ������
  // ���ĿǰToken���ĵ�һ��Tokenʱident��Ҳ���Ǳ�����
  // ���ｫ��������ֵ��e - ����outerDecl�����ı�����
  e.setName(tok.getValue());	/* use value as the variable name */
  // ������������浽ȫ�ֱ����б���Ա������������ʱʹ��
  addOuter(e);		/* add this variable */
  // �ڽ���﷨���ﴴ��һ�����������ڵ�
  emit.FieldDef(e);		/* issue the declaration */
  // �����ǰ�����ƥ�� [class] ����Ĳ��֣�������ǰ����static, 
  // extern ��Щ�ؼ��֣��������ϢҲ���浽���������ڵ���Ա� 
  // �������ɴ���ʱ�ο�
  if (e.getClassId() == Tok.T_DEFCLASS)
    e.setClassId(Tok.T_STATIC);	/* make sure it knows its storage class */

  // ���� outer_decl ������ { "," ident } ����������������
  // ���������ƣ�int a, b, c; �����ı����������
  // ���������ͨ���жϺ����Token�Ƿ��� ',' ����ɵ�
  /*
   * loop while there are additional variable names
   */
  while (io.getNextChar() == ',')
    {
    // ������� ','����ô������������ַ�
    tok.scan();
    if (tok.getFirstChar() != ',')
	io.Abort("Expected ','");
    // ����ɨ��
    tok.scan();
    // �����ҵ�һ��ƥ�� ident �����Token
    if (tok.getId() != Tok.T_IDENT)
	io.Abort("Expected identifier");
    // �ҵ�һ�������� - ��ƥ�� ident �����Token
    e.setName(tok.getValue()); /* use value as the variable name */
    // ������µ�ȫ�ֱ�����ӵ�ȫ�ֱ�������
    addOuter(e);		/* add this variable */
    // ��ȻҲҪ�ڽ���﷨���ﱣ��������������ڵ�
    emit.FieldDef(e);		/* issue the declaration */
    if (e.getClassId() == Tok.T_DEFCLASS)
      e.setClassId(Tok.T_STATIC);	/* make sure it knows its storage class */
    }

  // ������ǰ��ı������壬����������ŵ��ַ��ǲ��Ƿֺ� - ';'
  /*
   * move beyond end of statement indicator
   */
  tok.scan();
  if (tok.getFirstChar() != ';')
    io.Abort("Expected ';'");
  // ˳��������һ����䣬ɨβ����
  CommentFill();
  tok.scan();
#if DEBUG
  Console.WriteLine("declOuter2 token=["+tok+"]\n");
#endif
  }
```
