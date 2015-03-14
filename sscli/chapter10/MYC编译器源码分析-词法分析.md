�ʷ�����
=======

�ʷ������Ĺ�������Tok�ദ���乹�캯������һ��Io�������ļ�����������Tok���캯����Դ�룺

```CSharp
public Tok(Io ihandle)
{
    io = ihandle;
    // ��ʼ��Token���ַ����ࣩ�ֵ�
    InitHash();			// initialize the tokens hashtable
    // �����ļ��ĵ�һ���ַ�
    io.ReadChar();
    // ���ɨ���ļ�����ַ�����ȡ
    // ��һ���ַ����ࣨToken��
    scan();
}
```

���캯���е�һ����������InitHash��Ŀ���ǽ��ؼ��ֺͲ����������ɸ�����ʶ����ַ�����ʶ��� - Token����������Ŀ����Ϊ�˱����﷨������parser�������磬������������C��䣺

```CSharp
int foo(int a)
```

�������﷨������ȥ����������ַ����ʷ��������������ǽ�ȥ����һ�����������������ĸ�ʽ��

```
T_INT T_IDENT ��(�� T_INT T_IDENT ��)��
```

��ΪT_INT��T_IDENT����һ�������ͳ���������(�������ĵ����ַ�Ҳ���Ե��������ͳ����Դ��������﷨�������ڷ����﷨��ʱ�����������Щ��������InitHash�������ѱ�����������еĹؼ��ֺͶ��ַ��������������Ƹ�ֵ������ <<=�������������ͱ�ʶ�ţ�Token������Tok�����scan()����ɨ��Դ�ļ�ʱ������һ������ֵ����ѯ�ؼ��ֵı�ʶ�ţ�

```CSharp
public void InitHash()
{
    // Ϊ�ַ�����ʶ��Ŷ��ձ� �C tokens����ռ�
    tokens = new Hashtable();
    AddTok(T_LEFT_ASSIGN,	"<<=");     
    // ... ...
    AddTok(T_IF,		"if");
    // ... ...
    AddTok(T_STATIC,		"static");
    AddTok(T_INT,		"int");
    // ... ...
}
```

����Ӧ��ÿ����ʶ�ţ�Token���Ķ��壬�������Tok.csԴ�ļ����������ҵ���

```CSharp
public const int T_LEFT_ASSIGN	= 10001;
// ... ...
public const int T_IF			= 20001;
// ... ...
public const int T_STATIC		= 30002;
// ... ...
public const int T_INT		= 40003;
// ... ...
public const int T_IDENT 		= 50001;
public const int T_DIGITS 		= 50002;
public const int T_UNKNOWN		= 99999;
public const int T_EOF	 	= -1;
```

�ַ�����ʶ��Ŷ��ձ��ʼ����Ϻ��﷨�������Ϳ��Ե���Tok�����scan���������﷨�����ˣ�scan����ÿ��ֻ��������һ���ַ����ͣ�

```CSharp
public void scan()
{
    // ����ע�͡����з����ո���ַ�
    skipWhite();
    // ���жϵ�ǰ��ȡ���ַ��ǲ���һ����ĸ
    // �������ĸ��ͷ�Ļ���Ҫô�ǹؼ��֣�
    // Ҫô���Ǳ�����
    if (Char.IsLetter(io.getNextChar()))
      // ���ɨ�������ַ���ֱ��ʶ����ؼ���
      // ���߱�����Ϊֹ���˳�
      LoadName();
    // �����ǰ���ַ��� 0 - 9������
    else if (Char.IsDigit(io.getNextChar()))
      // ɨ�����������ֲ�����
      LoadNum();
    // ����ǲ�������ɨ�������Ĳ������ַ���
    else if (isOp(io.getNextChar()))
      LoadOp();
    // ����ļ��Ѿ���ȡ�����
    else if (io.EOF())
      {
      // ���������ʶ��� T_EOF����ʾ�ļ���ȡ���
      value = null;
      token_id = T_EOF;
      }
    else
      {
      // ����ַ�����һ���Ϸ����ַ��������T_UNKNOWN
      // T_UNKNOWNû�б��κ��﷨����
      // ����﷨��������ɨ���﷨�Ĺ�����
      // �������ʶ��������п�����Դ�������﷨����
      value = new StringBuilder(MyC.MAXSTR);
      value.Append(io.getNextChar());
      token_id = T_UNKNOWN;
      io.ReadChar();
      }
    skipWhite();
    // �������룬�����myc.exe�ǵ��԰汾��������������
    // ��ӡ����ǰʶ����ַ����ͣ�����myc.exe�Ŀ������Ŵ�
#if DEBUG
    Console.WriteLine("[tok.scan tok=["+this+"]");
#endif
}
```

scan������Tok����������ĵĺ�������ʵ���������ǰ��myc�﷨����Щ�ʷ����򣨻��������Ĺؼ��ֺͲ�����ʶ�𣩣�

```
letter ::= "A-Za-z";
digit ::= "0-9";

name ::= letter { letter | digit };
integer ::= digit { digit };
```

������ͨ��˵��LoadName���������ʹʷ�������ϸ�ڣ�

```CSharp
void LoadName()
{
  // �����ȡ�����ַ�
  value = new StringBuilder(MyC.MAXSTR);
  skipWhite();  
  // ������֤ - ȷ����һ���ַ�����ĸ
  if (!Char.IsLetter(io.getNextChar()))
    throw new ApplicationException("?Expected Name");
  // ������ŵ��ַ�ֻ�������ֻ�����ĸ
  while (Char.IsLetterOrDigit(io.getNextChar()))
    {
    // �����ַ����Ա��ж��Ǳ����������ǹؼ���
    value.Append(io.getNextChar());
    // ��Դ�ļ����ȡ��һ���ַ�
    io.ReadChar();
    }
  // ���ַ�����ʶ������ѯ��ȡ���Ĵ����ǲ��ǹؼ���
  token_id = lookup_id();
  // ���ǹؼ��ֵĻ�����ô���Ǳ���������������
  if (token_id <= 0)
    token_id = T_IDENT;
  skipWhite();
}
```

��������Ͼ��Ǵʷ������Ĺؼ������ˣ�������˵����ʱ�������������˹��캯���� io.ReadChar()���������������������������Ͽ��Ƕ�ȡһ���ַ�����ʵ���ϴ�Դ�ļ�һ���ַ�һ���ַ��Ķ�ȡЧ��ʵ����̫���ˣ����һ�㶼�Ǵ�Դ�ļ����ȡһ����ַ����������ڴ�����Ч�ʣ�

```CSharp
// Io.cs �C ReadChar����

public void ReadChar()
{
  // �ж��ǲ��Ƕ����ļ�ĩβ��
  if (_eof)			// if already eof, nothing to do here
    return;
  // ������滹û��ʵ���������߻�������ַ�
  // �Ѿ���������ˣ�����һ���µĻ���
  // �����ϵĻ������飬�����������ջ��ƴ���
  if (ibuf == null || ibufidx >= MyC.MAXBUF)
    {
    ibuf = new char[MyC.MAXBUF];
    _eof = false;
    // ��Դ�ļ����ȡһ������ݵ�������
    ibufread = rfile.Read(ibuf, 0, MyC.MAXBUF);
    ibufidx = 0;
    if (buf == null)
      buf = new StringBuilder(MyC.MAXSTR);
    }
  // �ӻ������ȡ��һ���ַ�
  look = ibuf[ibufidx++];
  // �ж���ζ�ȡʱ���Ƿ��Ѿ���Դ�ļ�ĩβ��
  if (ibufread < MyC.MAXBUF && ibufidx > ibufread)
    _eof = true;

  /*
   * track the read characters
   */
  // ���浱ǰ��ȡ���ַ����Ա�������ILԴ�ļ���ʱ��
  // ���԰�CԴ������ɵ�ILԴ���Ӧ����
  buf.Append(look);
  // ����������У������кţ��к��ڱ����﷨����
  // ��ʱ����õ�����֪�����﷨������кű���
  // ����Ա�ҵ�����
  if (look == '\n')
    bufline++;
}
```

��Io.ReadChar������ᱣ���ȡ��CԴ�룬��Ҫ����ILԴ�ļ���ʱ�������Ϣ��������C����IL���Ķ�Ӧ��ϵ������������������myc���Դ��Ĳ���Դ���ļ���
 
![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter10/myc-list-cmd.png)

Ч������ͼ��

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter10/myc-list-result.png)
 
