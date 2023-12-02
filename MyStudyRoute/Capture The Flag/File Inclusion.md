==Local Flie Inclusion==
关于‘.’：
etc/dic/.将会是etc/dic/（当前目录）
etc/dic/..将会是代表etc/（上一目录）

include函数可以通过%00截断来忽略掉一些已经写死在字符串后面的字符。如include($\_GET\['para'] . "php") 加入 %00截断来去掉后面的php后缀来包含其他文件。ps:_在PHP5.3.4已修复


如果过滤了“../”可以尝试双写：
....//....//....//....//
过滤后将是：../../../../
![[Pasted image 20231118213704.png]]

一些常见的目录：
![[Pasted image 20231118214506.png]]
![[Pasted image 20231118214523.png]]
==Remote File Inclusion==
进行RFI时，可以使用python3 -m http.sever 端口
快速部署一个python3 http服务器来托管要RFI的文件。