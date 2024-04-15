sqli:爆多字段别忘了用group_concat
爆字段别忘了指定数据库以及表[[EXAM特供WP#^1e5e69]]
fileinclusion:写日志！伪协议！
日志：
```
/var/log/apache2/access.log
/var/log/nginx/access.log
/var/log/httpd/access.log
```
伪协议：
```
php://filter/read=convert.base64-encode/resource=
php://input
data://text/plain;base64,PD9waHAgZWNobyBwaHBpbmZvKCk7Pz4=
file:///etc/passwd
phar://压缩包名/内部文件名
zip://压缩包绝对路径#内部文件名（```
rce:遇到2>&1代表输出错误，输出错误的一律用反引号将命令结果输出到错误中。
遇到无回显的就用重定向到文件内进行查看结果，活用通配符
想象tab补全的内容可能会输出有趣的内容。tab->%09 回车->%0a
过滤特定词汇的使用不存在的变量绕过p${9}hp -> php