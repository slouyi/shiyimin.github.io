ǰ����ʾ�˱���SSCLI����ķ�������Windows�£���

 1. �ڡ�Visual Studio 2005 Command Prompt���£�����SSCLI�ĸ�Ŀ¼��
 2. ���� env.bat �ű�׼��������
 2. ���� buildall.cmd �ű���ʼ������̡�

env.bat�����˵�ǰSSCLI�����л�����������﷨�ǣ�`env [option]`������[option]������ *debug*��*checked* �� *free* ����������ѡ��˵�����±�

* **debug**���رմ����Ż����ã����õ����ô��루һ����ͨ���������뿪�أ������ɵ����÷����ļ���
* **checked**�� �򿪴����Ż����ã����������ô��룬���ɷ����ļ���
* **free** ���򿪴����Ż����ã��رյ����ô��룬���ɷ����ļ���
���ڽ���������Ҫ��һ�㶼��ѡ��debugѡ�

�趨�����л�����buildall.cmd�ı�����̿��Է�Ϊ���漸���裺

1. ����������������߼���ƽ̨����㣨PAL���ͷ��йܳ���unmanaged�����빤�ߣ�
1. ���븨�����������߼���ʣ��Ĺ��߼�����ػ����ܹ���
1. ����CLR�ںˡ�������⣬C#������������֧�ֹ��ߣ�
1. ��������.NET����֧�ֹ���
1. ���������йܳ���ı���������JS��������

��ʵ.NET�������ǿ�ƽ̨�ģ�SSCLI 2.0����֧��Windowsƽ̨����֧��FreeBSD��Mac OS X��Դ�������FreeBSD 4.8��Mac OS X 10.2�±���ͨ����ͨ���޸�һЩԴ�룬��������Linuxƽ̨�±����ʹ�ã�ʵ����SSCLI 1.1ͨ��һЩ�޸Ŀ�����Red Hat 8.0�±���ͨ����

Ϊ���ں���˵�����㣬������SSCLI���õļ�������������ָ�������ᵽ���ļ���·����

* **%ROTOR_DIR%**�� SSCLI�ĸ�Ŀ¼����Windowsƽ̨��һ����`c:\sscli`���ڱ�������У�����ϵͳ���������ô˻���������
* **%_NTTREE%**������SSCLI�����������ı���·������ֵһ���ǣ�`%ROTOR_DIR%\binaries.x86*.rotor\`��*�Ÿ���bat ��ѡ�ֵ����env.bat��ѡ����`debug`����ô����ֵ���� `%ROTOR_DIR%\binaries.x86dbg.rotor\`��

�����������߼�
--------------------
SSCLI��Դ����C++��C#��Щ���йܺ��йܱ��������ɣ�����SSCLI�ǿ�ƽ̨�ģ�����ڱ�������б���ϵͳ���ò���ϵͳ�ϰ�װ��C++����������ñ���SSCLIʣ��Դ��Ĺ��߼�������һ����������У������������߼�����������Щ�����

* ƽ̨����㣨PAL����PAL��ϵͳ����������������������˱����ȱ�������

   * Դ��λ�ã�Windowsƽ̨����%ROTOR_DIR%\pal\win32
   * Դ��λ�ã�Unixƽ̨����%ROTOR_DIR%\pal\unix
   * ���·����%_NTTREE%\rotor_pal.dll

* Nmake���߼���nmake��Windows SDKϵͳ�µı��빤�ߣ�����unix�µ�make��SSCLI�Դ���nmake��Դ�룬�������Unix��Linuxϵͳ�±��룬�����˹��ߣ�������Windowsƽ̨�£�����SDKϵͳ���Դ���nmake�����ˣ�
  * Դ��λ�ã�%ROTOR_DIR%\tools\nmake
  * ���·����%_NTTREE%\nmake.exe

* �������Ĵ����ߣ�
  * Դ��λ�ã�%ROTOR_DIR%\tools\binplace
  * ���·����%_NTTREE%\binplace.exe

* ����ϵͳ��������ı��빤�߼���
  * Դ��λ�ã�%ROTOR_DIR%\tools\build
  * ���·����%_NTTREE%\build.exe

�����������߼�
--------------------
�����������߼���ǰ������������߼��������ǣ������������߼��õ���SSCLI����ϵͳ�Դ���build.exe����ģ�Դ�ļ��б�ȱ�������Ǳ�����sources.lst�ļ���ģ��������������߼�����make������룬����Դ�ļ��б�ȱ�������Ǳ�����makefile�ļ�����ġ�

�����������߼�����������Щ�����

* ��Դ������
   * Դ��λ�ã�%ROTOR_DIR%\tools\resourcecompiler
   * ���·����%_NTTREE%\resourcecompiler.exe

* PAL����ʱ��PAL RT����PAL RT������SSCLI����һЩ�������߶��õ��Ŀ�ƽ̨�Ĺ��ܡ�

  * Դ��λ�ã�%ROTOR_DIR%\palrt\src
  * ���·����%_NTTREE%\rotor_palrt.dll

����CLR�ںˡ�������⣬C#������������֧�ֹ���
--------------------------------------------------------------

�������߼�������ɺ󣬾Ϳ��Կ�ʼ��������CLR��.NET������⡢C#�������ȹ����ˣ������ߵ�Դ��λ�á����·�����±���ʾ��

* **\clr\src**��	����CLR�ں˺ͻ�������Դ��·��
* **\clr\src\vm**��CLR����������Դ�룬����GC��JIT��������������������
* **\csharp**��C#��������Assembly���ӳ����Դ��·��
* **\clr\src\bcl**��.NET�������Դ�룬��System.IO��System.Collections��Щ�����ռ���������Դ�붼������
* **\clr\src\dlls**��	�������̸�CLR����������ļ����ؼ�DLL��Դ�룬����������Щ�����
* **shim**����Ҫ��ȷ����ǰ�����ϰ�װ����Щ.net�汾
* **mscorsn**��������ǿǩ����֤
* **mscorpe**��Windowsƽ̨��ִ���ļ�PE��ʽ��д��
* **mscoree**�������ڽ����м���CLR�����
* **mscordbi��mscordbc**�����Է���
* **\src\utilcode**�����ܶ�CLR�������������ͨ�ô���
* **\src\fjit**��JIT����������mscorejit.dll��Դ��
* **\src\fusion**�������ͼ���Assembly��GAC��Global Assembly Cache�����������fusion.dll��Դ��
* **\src\ilasm**��IL���Ա�����
* **\src\ildasm**��IL�����빤��
* **\src\debug**���йܵ����� cordbg ��Դ��
* **\src\md**��AssemblyԪ���ݶ�д��
* **\src\tools**��������йܹ��ߵ�Դ�룺
* **clix**���йܳ������ִ�й���
* **ildbsymbols**���йܳ�����Է����ļ���д����
* **metainfo**��assemblyԪ���ݶ�ȡ����
* **peverify**����֤�йܳ����IL����
* **internalresgen**��
* **sn**��ǿǩ����������
* **permview**��AssemblyȨ�޲鿴����
* **gac**��ȫ��Assembly���������
* **sos**��Windbg����.NET����ĸ���������

����.NET����֧�ֹ���
-------------------------------
�������ṩ��һЩ���������������������Դ�룺

* **System.dll**��%ROTOR_DIR%\fx\src\sys
* **System.xml.dll**��%ROTOR_DIR%\fx\src\xml
* **System.Runtime.Serialization.Formatters.Soap.dll**��%ROTOR_DIR%\managedlibraries\soapserializer
* **System.Runtime.Remoting.dll**��%ROTOR_DIR%\managedlibraries\remoting

�����йܳ���ı�����
----------------------------

SSCLI���滹������һ��ʹ��C#���Կ�����Microsoft Jscript������ʵ�֣������������������ܱ����йܳ�����Դ��·���ǣ�`%ROTOR_DIR%\jscript`