# ׼��CLRԴ���Ķ�����

΢������CLR 2.0��Դ�룬���Դ���ǿ���ֱ����freebsd��windows�����±��뼰���еģ�����΢�� [shared source cli](http://www.microsoft.com/en-us/download/details.aspx?id=4917) ���Ӵ����أ�����7zip�ȹ��߽�ѹ���Ժ���sscli �C ��Shared Source CLI��

��ѹ�󣬸�Ŀ¼����readfirst.html�ļ�������˵���˸ÿ�Դ�汾������Ĺ����б�

1. ���͵�ʵ�֣�
1. �������Ĵ������ɣ�
1. ί�е�ʵ�֣�
1. ���䣻
1. װ�����Assembly����Ԫ���ݸ�ʽ��
1. ����������ί�У�
1. .NET������⣨BCL����
1. ��������д���

Ҫ����clrԴ��Ļ�����Ҫ��������������

* Microsoft Visual Studio 2005רҵ�����ϣ���ʹ��Ĭ�ϰ�װ�������ڱ����ʱ������Ҳ����ļ��������
* Perl��

���⣬����һ��׼�����裬������������İ��Windowsϵͳ�ж�������������Rotor�в���Դ������ANSI�ַ���ţ������к�����936 Code Page��Ҳ����Simplified Chinese GBK��չ�ַ������޷��������ַ�����Build��ʱ��VC������CL�ᱨ ***warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss*** ��ͬʱ��Build��ʱ�����ڴ���/WX���أ��κ�warning���ᱻ������error��ֱ�ӵ��²��ֱ���ʧ�ܡ���������У�

1. ��ȫ���������Դ����ת����Unicode���룻
1. ����ϵͳ��ǰ���������ã�Locale��ΪӢ��

һ����˵��ϵͳ��Locale����ʡ��ڡ�������塱�С����ڡ�ʱ�䡢���Ժ��������á��еġ����������ѡ��ġ��߼���ҳ���޸ġ���Unicode��������ԡ���ѡ��Ϊ��Ӣ�ģ��������� ���������ɡ�
 
�ڱ�������У���Ҫ�������������PATH·�����Ա����������ҵ����ǣ�
 
��Perl��·�������������磺`C:\Perl\bin`��

��װ��������������sscliԴ���ѹ֮�󣬴򿪡�Visual Studio 2005 Command Prompt�����ڣ��л���sscli�ĸ�Ŀ¼����������Ŀ¼·���ǣ�c:\sscli������ִ����������

```
cd /d c:sscli
rem ���õ�ǰ��������л���Ϊ���Ի���
env debug
rem �������еĳ���
buildall
```��

����ɹ�֮��дһ���򵥵�C#�ļ������±�

```
using System;
 
public class Hello
{
      public static void Main()
      {
             Console.WriteLine("Hello, sscli");
      }
}
```��

�ڱ���CLRԴ��Ŀ���̨�����������������ִ��C#����ǰ��ִ�е� env debug����Ѿ��Զ����ú�PATH����������C#������csc.exe����ָ��������Ǳ���õĳ��򣩣�

1. ���룺csc test.cs
1. ���г���clix test.exe

** ע�� **

1. ����ʹ��Ӣ��ԭ���Windows XP���б��룬���߰�ǰ�����ĳ�Ӣ�ĵ��������ã�
1. ��Ҫʹ��VS 2005���ϵ�IDE���룬��������VS 2008����ɹ�������д��ƪ���µ�ʱ���������ܶ�������Ϊ��ʡ�µĻ���������VS 2005���룻
1. ��Ҫȷ��VS 2005��װ���Ժ��� `C:\Program Files\Microsoft Visual Studio 8\VC\PlatformSDK` ����ļ��У��ڱ����ʱ����Ҫ�õ������ͷ�ļ��Ϳ��ļ���