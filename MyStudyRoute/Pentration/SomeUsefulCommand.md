Powershell下载
```
Invoke-WebRequest "[IP:PORT]" | Select-Object -ExpandProperty Content | Out-File "[FILE NAME]"
```
检查.NET Framework版本
```
reg query "HKLM\SOFTWARE\Microsoft\Net Framework Setup\NDP" /s
```
hello Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbosehttps://github.com/Restonweb/ObsidianStorge.git