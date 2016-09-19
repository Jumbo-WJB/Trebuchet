# Trebuchet
#####MS15-076 (CVE-2015-2370) Privilege Escalation
######Copies a file to any privileged location on disk

Compiled with VS2015, precompiled exe in Binary directory

Usage: trebuchet.exe C:\Users\Bob\Evil.txt C:\Windows\System32\Evil.dll

This is a lightly modified Proof of Concept by James Forshaw with Google, found here: https://code.google.com/p/google-security-research/issues/detail?id=325

CreateSymlink tool was written by James Forshaw found here:
https://github.com/google/symboliclink-testing-tools

Notes:
 - Microsoft.VisualStudio.OLE.Inerop.dll must be in the same directory
 - Exploit can only be one once every 2-3 minutes. This is because RPC can be held up by LocalSystem
 - The destination file can't already exist
 - Tested on x64/x86 Windows 7/8.1

Trebuchet.exe D:\1.dll c:\windows\lpk.dll
将漏洞利用的方式改为向目标主机磁盘的任意位置写入任意文件。
幸运的是,Forshaw还对符号链接以及通信链接等方面进行过大量的研究,而这些方面的知识对于漏洞利用来说是非常有帮助的。我们在这里给大家提供了他在2015 Syscan信息安全大会上进行演讲时所用的演示文稿,以及他的GitHub代码。
现在,我需要从漏洞信息中提取出非特权文件的符号链接策略。从本质上来说,它会使用一个通信链接,并且将其与符号链接进行绑定,然后将所有参数写入一个全局命名空间之中,即\RPC Control\,并在没有获得系统管理员权限的情况下得到C:\Folder\FileA指向C:\FileB的文件指针。让我们来仔细看看它是如何将文件入‘C:\Windows\System32\Evil.dll’的:
1.    在‘C:\Windows\Temp\{Random}’ 与‘C:\Users\Public\Libraries\Sym’之间建立一条目录链接;
2.    在‘\??\C:\Users\Public\Libraries\Sym’与‘\RPC Control\’之间建立一条新的目录链接;
3.    在‘\RPC Control\ (2)’与‘\??\C:\Windows\System32\Evil.dll’之间建立一条符号链接;
4.    在漏洞利用的过程中,系统会尝试向‘C:\Windows\Temp\{Random}/’目录中写入一个文件,并将其指向‘C:\Windows\System32\Evil.dll’;
你需要注意的是:在CreateSymlink项目中,步骤2和步骤3是会同时进行的。
通过上面的操作来修改PoC代码,我们就可以将任何一个文件拷贝到一个有特殊权限的位置。我们还将最终的成品命名为’ Trebuchet’。



你也许会认为:“那又怎么样?你只是可以写入任意文件而已。“我们将漏洞利用的方法提供给你了,你要如何使用这些方法完全取决于你们自己,但是如果你对DLL劫持比较熟悉,那么权限提升对你来说也并不会非常的困难。
POC:
https://github.com/monoxgas/Trebuchet
值得注意的是:
在每2-3分钟之内,这个漏洞利用方法只能使用一次。否则远程调用请求会被目标主机的本地系统挂起。
目前,Interop DLL必须与漏洞利用文件处于同一目录下。
这个漏洞利用方法只在Windows 7/8.1的32位以及64位系统平台上进行过测试。
所有的许可证代码属于谷歌公司或James Forshaw个人所有。
项目中绝大多数的代码都可以被优化或简化。
在此,我们要感谢James Forshaw给我们提供的分析材料,以及他在研究中所付出的努力。
