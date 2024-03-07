# 执行摘要
使用NMAP扫描，知晓以下端口开放：
![[Pasted image 20240307181708.png]]
80端口运行IIS服务器，135RPC服务，139，445表示其SMB服务开启，3389运行了RDP服务。
访问80页面是IIS的默认页面，仍使用GOBUSTER进行快速目录扫描
尝试访问其SMB服务：
![[Pasted image 20240307182940.png]]
发现四个share
尝试匿名访问nt4wrksv：
![[Pasted image 20240307183039.png]]
可以匿名访问，且其中有名为“passwords.txt”的文件
下载打开：
![[Pasted image 20240307183229.png]]
base64解码：
`Bob - !P@$$W0rD!123
`
# 漏洞
# 利用评估
# 补救建议
