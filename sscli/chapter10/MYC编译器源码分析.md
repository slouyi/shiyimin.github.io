Դ�����
----

��������������C#����д�ģ���ͼ�г���MyC����������һ��CԴ�ļ��Ĺ��̣�������·�����£�

1.	���������Main�����������������в�������ȡԴ�ļ�������ʼ������̡�Main������MyC.cs�ļ�����IO.cs�ļ���Ҫ�����ȡԴ���ļ�����ز������±���Main������Դ�루��ע��ע�͵ķ�ʽ��ʾ����IO.cs�ļ��õ�����һ��С��˵����

``` CSharp 
public static void Main()
{
    try
{
��// ��Դ��ע�ͣ�������99��д�ģ�Ҳ����˵.NET�������ڿ�����
    // �����Ǹ�ʱ���������û��������Main�������������в����Ŀ���
    // ��������������ⷽ����ȡ����������в���
        String[] args = Environment.GetCommandLineArgs();
        // ��ʼ����ȡԴ�ļ���IO���󣬸ö�����Դ�ļ����ֽ����ķ�ʽ
        // �������һ������ �C �ʷ�����
        Io prog = new Io(args);
        // �ʷ��������󣬸ö���Ĺ����ǹ��˵�Դ���в���Ҫ���ַ�������ո�
        // ע��֮��ģ����Ұ�Դ���е��ַ����� �C Tokenize���Ա��﷨������
        // ������Ľ����﷨
        Tok tok = new Tok(prog);
        // �﷨�������󣬽�����Ϻ��Ǵ������ɽ׶Σ���һ���﷨��������
        // ��ֻ�������﷨������������ƿ��ԶԽӶ��ֽ���ļ�����ֶΡ�����
        // ˵�����������ɿ�ִ���ļ���exe.cs������ILԴ���asm.cs����ͨ��
        // �����﷨����ʹ�ò�ͬ������������ɽ���ļ���
        Parse p = new Parse(prog, tok);
        // �����Զ����µķ�ʽ�����﷨����
        p.program();
        // ���빤���Ѿ���ɣ��رմ򿪵�Դ�ļ��������Դ
        prog.Finish();
    }
    catch (Exception e)
{
    // ������������κδ��󣬼��жϴ�����ӡ������Ϣ���˳�����
        Console.WriteLine("Compiler aborting: " + e.ToString());
    }
}
```

2.	MyC���﷨�ܼ򵥣���˱�������Ǻܸɾ��Ĵʷ��������﷨�������������ɺͽ������Ĺ��̡����дʷ�����������tok.cs���﷨����������parse.cs�У�Emit.cs����������ɣ���Asm.cs��Exe.cs�ֱ���������в��������ã����������յĿ�ִ���ļ���ѡ���Ƿ����ILԴ�롣
IO.cs - IO����
��Դ�ļ���ȡ���ڴ棬��������ʽ����Ĵ��붼����IO��������棬IO�Ĺ��캯�����������в���������Դ�ļ����ȴ�Tok.cs��������ָ�Դ�ļ����ַ�һ���������ڴ沢�������������Ĺ��캯����Դ�룺

``` CSharp
public Io(String[] a)
  {
  int i;

  args = a;
  // ���������в����������ݲ������ڲ��Ŀ��ƿ��أ������뿴�����ParseArgs
  // ������Դ����
  ParseArgs();

  // ��Ҫ�����Դ�ļ�
  ifile = new FileStream(ifilename, FileMode.Open,
			 FileAccess.Read, FileShare.Read, 8192);
  // ���Դ�ļ������ڣ������˳�
  if (ifile == null)
    {
    Abort("Could not open file '"+ifilename+"'\n");
}
  // ������ʽ����ʽ��ȡԴ�ļ�
  rfile = new StreamReader(ifile); // open up a stream for reading
 
  // ����Դ�ļ��������趨�������ļ����ļ���
  i = ifilename.LastIndexOf('.');
  if (i < 0)
    Abort("Bad filename '"+ifilename+"'");
  int j = ifilename.LastIndexOf('\\');
  if (j < 0)
    j = 0;
  else
    j++;

  classname = ifilename.Substring(j,i-j);

  // ���������в�������������.exe��.dll�ȿ�ִ���ļ��������������
  // ILԴ���.lst�ļ� 
  if (genexe)
    ofilename = classname+".exe";
  if (gendll)
    ofilename = classname+".dll";
  if (genlist)
{
// �����Ҫ���ILԴ�룬��Ϊԭ���Ŀ�ִ���ļ�ҲҪ�������Ҫ����һ���µ��ļ�
    lst_ofilename = classname+".lst";
    lst_ofile = new FileStream(lst_ofilename, FileMode.Create,
			 FileAccess.Write, FileShare.Write, 8192);
    if (lst_ofile == null)
      Abort("Could not open file '"+ofilename+"'\n");
    lst_wfile = new StreamWriter(lst_ofile);
    }
  }
```

����������IO���ﴦ�������в����ģ���������ʵ������һЩ�ַ�������Ļ���Ľ����¹ؼ����룺

``` CSharp
void ParseArgs()
  {
  int i = 1;
  
  // ����������Ҫ�������������������������ֲ��˳�
  if (args.Length < 2)
    {
    Abort("myc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
    }
  
  // ������������в���
  while (true)
    {
    if (args[i][0] != '/')
      break;
    // ���� /? �������������������ı�
    if (args[i].Equals("/?"))
      {
      Console.WriteLine("Compiler options:\n  myc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
      Environment.Exit(1);
      }
// ����� /debug ����������ڲ��� gendebug ���أ���������ڴ������ɵĹ���
// �л��õ�
    if (args[i].Equals("/debug"))
      {
      gendebug = true;
      i++;
      continue;
      }
// ... ... �������ƵĴ���
// ����� /outdir ���������ȡ��������ָ����Ŀ¼·��
    if (args[i].Length > 8 && args[i].Substring(0,8).Equals("/outdir:"))
      {
      genpath = args[i].Substring(8);
      i++;
      continue;
      }
// ǰ����ô���if�൱��switch �� case �� default ������� case ����·��
// ������δ��뼴�� default ����·�� �C ��������в�������ǰ���if����
// ����ִ������� continue �Ӿ�����ѭ������ִ�е����˵������
// ���޷�ʶ��Ĳ�������˱�������˳�ִ��
    Abort("Unmatched switch = '"+args[i]+"'\nArguments are:\nmyc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
    }

  // ���ǰ���ѭ��ִ����ϣ����в����б�δ����˵�������˲�֧�ֵĲ���
  if (args.Length-i != 1)
    {
    Abort("myc [/debug] [/nodebug] [/list] [/dll] [/exe] [/outdir:path] filename.myc\n");
}

  // ���һ��������Ҫ�����Դ�ļ�·��
  ifilename = args[args.Length-1]; // filename is last
  }
```

IO���д󲿷ֺ�������ΪTok.cs����ģ�������������ڽ��ʹʷ�������ʱ��˵��
