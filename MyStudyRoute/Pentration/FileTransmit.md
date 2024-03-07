内网间可以使用Python的http模块，如未安装Python，使用curl之类可以获取内容的命令也可获取。
Windows没有WGET，使用：
`Invoke-WebRequest "http://[IP:PORT]/[File]" | Select-Object -ExpandProperty Content | Out-File "[File Name]"`作为alias。
Windows可以使用SMB共享，linux使用SMBclient模块
现成的Git库可以直接clone。