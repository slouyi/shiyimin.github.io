# ��SSCLI�е����йܴ���

SSCLIֻʵ���˲���.NET���Թ��ܣ����Ҳ�֧����Visual Studio��ֱ�ӵ���SSCLI������ִ�е��йܳ�����SSCLI��ֻ�������Դ���cordbg.exe����SSCLI�����µ��йܳ���

��Ϊ.NET Framework 2.0 SDKҲ�Դ���cordbg.exe��������Ҫ��SSCLI�������ִ���� `env.bat` �����ʼ������SSCLI�����������ô����cordbg�汾����SSCLI����

���ǽ����������C#��������ʾcordbg���÷�������SSCLI�Դ������ӣ�Դ��λ�ڣ�`samples\howto\basedatatypes\stringformat`��

```C#
using System;
using System.IO;
using System.Globalization;

public enum Color
{
    Red = 25,
    Blue,
    Green
}//enum Color

public class stringFormat
{
    public static void Main(string[] args)
    {
        DateTime d = DateTime.Now;

        PrintFormat(d, "d");

        Console.WriteLine("AbbreviatedDayNames = new string [] {Sund, Mond, Tues, Weds, Thur, Frid, Satu}");
        DateTimeFormatInfo dtfi = new DateTimeFormatInfo();
        dtfi.AbbreviatedDayNames =
                   new string[] { "Sund", "Mond", "Tues", "Weds", "Thur", "Frid", "Satu" };
        PrintFormat(d, "ddd", dtfi);
        Console.WriteLine("------------------------------------------------------------------------------\n");

        // Enum formatting
        Console.WriteLine("Enum Formatting");
        Console.WriteLine
          (
          "Name: {0}, Enum Value: {1}, Enum Value Hex: {2}",
          Color.Green.ToString("G"),
          Color.Green.ToString("D"),
          Color.Green.ToString("X")
          );
        Console.WriteLine();

        //Number standard formats
        double num = System.Math.PI * -3.0;
        PrintFormat(num, "c4");
    } //Main()

    public static void PrintFormat(IFormattable inputData, string formatString)
    {
        PrintFormat(inputData, formatString, null);
    }//PrintFormat()

    public static void PrintFormat(IFormattable inputData,
                                   string formatString,
                                   IFormatProvider provider)
    {
        try
        {
            if (provider == null)
            {
                Console.WriteLine("{0}\t{1}",
                                  formatString,
                                  inputData.ToString(formatString, provider));
            }//if
            else
            {
                string formstr = "{0} in the custom " + formatString + " format is {1:" + formatString + "}";
                Console.WriteLine(string.Format(provider, formstr, inputData, inputData));
            }//else
        }//try
        catch (Exception e)
        {
            Console.WriteLine("Exception in PrintFormat(): {0}", e.Message);
            Console.WriteLine("Data was {0}, Format string was: {1}, Type was {2}",
                               inputData,
                               formatString,
                               inputData.GetType().Name);
        }//catch
    }//PrintFormat
}//class stringformat
```

�����ʱ����Ҫ��C#�������ġ�/debug�����Կ��ز������ɵ��԰汾�ĳ��򣬱��ڵ�������cordbg�����Գ���

`csc /debug stringformat.cs`

ע�⣺csc����ҲҪ�ڳ�ʼ��SSCLI����֮��������У�����Ҳ���õ�.NET Framework SDK�Դ���csc���

����ִ�����֮�󣬻ῴ���������������stringformat.exe��ִ���ļ����⣬��������stringformat.ildb��������õķ����ļ���

ִ����������ʼ���ԣ�

`cordbg stringformat.exe`

cordbg���н�������ͼ��

![](https://github.com/shiyimin/shiyimin.github.io/blob/master/images/sscli/chapter1/cordbg-interface-overview.png)

����Visual Studio��ЩIDE���Գ���ͬ��cordbgһ��ʼ������ֱ�����б����Գ��򣬶������жϳ����ִ�� �C ���ڵ�15��Main��������ڴ��жϣ��ȴ��������롣������Ϊ��Ҫ����ʱ���ÿ�����Ա׼�����Թ��� �C �����öϵ�֮��ġ��������������롰help��������Բ鿴cordbg����õ������һƪ���½�����cordbg�������÷���

����������ʾ��������������±�������ע����#�ſ�ͷ�Ĵ������壩˵��ÿ����������壺

```bash
C:\sscli20\research\chapter1>cordbg stringformat.exe
Microsoft (R) Common Language Runtime Test Debugger Shell Version 2.0.50826.0
Copyright (c) Microsoft Corporation.  All rights reserved.

#
# run �����������������Գ������ִ��cordbg����ʱ�������˱����Գ����ļ��Ĳ�����
# ����Զ�ִ���������
#

(cordbg) run stringformat.exe
Process 1588/0x634 created.
[thread 0xe4] Thread created.

#
# ��ǰ�����жϵ�λ�ã�ǰ��� 015 ��ʾ��ǰ�жϵ�λ���ǵ�15��
#

015:     {

#
# ��Դ��ĵ�18�����öϵ�
#

(cordbg) break 18
Breakpoint #1 has bound to C:\sscli20\research\chapter1\stringformat.exe.
#1      c:\sscli20\research\chapter1\stringformat.cs:18 Main+0x7(il) [active]

#
# �ָ������ִ��
#

(cordbg) g
break at #1     c:\sscli20\research\chapter1\stringformat.cs:18 Main+0x7(il) [ac
tive]

#
# �����˶ϵ㲢�ж�ִ�г���
#

018:         PrintFormat(d, "d");

#
# �鿴�ֲ�����d��ֵ
#

(cordbg) p d
d=<System.DateTime>
  dateData=0x88d21f5af1505bc5
  
#
# �鿴���к����������ֲ�������ȫ�ֱ���
#

(cordbg) p
args=(0x00d4217c) array with dims=[0]
CS$0$0000=<null>
d=<System.DateTime>
dtfi=<null>
num=0
$result=(strong handle, Size: 4) (0x00d43fe0) <System.Reflection.TargetParameter
CountException>
$thread=(0x00d42830) <System.Threading.Thread>

#
# ����һ����������Ҫ���������������͵�ȫ�� �C �����������ռ䣬
# �����ͺ�����֮���á�::���ָ�������ͨ���ո�ָ�
#

(cordbg) funceval System.DateTime::ToString d
There are 4 possible matches for the method ToString. Pick one:

#
# ��Ϊ�к������صĹ�ϵ������cordbg���г���ѡ�ĺ��������ÿ�����Աָ��
# ��Ҫ���õĺ�����
#

0) none, abort the operation.
1) [060002eb] String ToString()
2) [060002ec] String ToString(String)
3) [060002ed] String ToString(IFormatProvider)
4) [060002ee] String ToString(String, IFormatProvider)

Please make a selection (0-4): 1
Function evaluation complete.
$result=(strong handle, Size: 4) (0x00d5b6a0) "2015/2/25 21:41:26"
018:         PrintFormat(d, "d");

#
# funceval���ú����Ľ�������浽һ���� $result �ı������ִ�к�����funceval
# ����ʱ������������һ������ĵ��ý��
#

(cordbg) funceval System.Console::WriteLine $result
There are 19 possible matches for the method WriteLine. Pick one:
0) none, abort the operation.
1) [060006d4] Void WriteLine()
... ...
14) [060006e1] Void WriteLine(String)
... ...
19) [060006e6] Void WriteLine(String, Object[])

Please make a selection (0-19): 14
2015/2/25 21:41:26
Function evaluation complete, no result.

018:         PrintFormat(d, "d");

#
# k����������ֹ��ǰ����ִ�еı����Խ���
#

(cordbg) k
Terminating current process...
Process exited.

#
# exit�����˳�cordbg����
#

(cordbg) exit
```