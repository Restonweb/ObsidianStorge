[BUUCTF] WEB PHP1
	==PHP反序列化漏洞==
	绕过函数 *PHP序列化与反序列化，对象属性值修改绕过*
	扫描文件，得到源码，分析函数作用。
[ACTF2020 新生赛]Include
	==Include文件包含漏洞==
	PHP伪协议相关中的php://协议
	php://filter用于读取源码(以base64编码输出)：
	一些敏感信息会保存在php文件中，直接打开是不会将它们显示在界面上的
	但是经过base64编码再解码就可以得到其完整的源码信息：
	php://filter/convert.base64-encode/resource=文件路径
	php://input用于执行php代码：
	可以访问请求的原始数据的只读流, 将post请求中的数据作为PHP代码执行。当传入的参数作为文件名打开时，可以将参数设为php://input,同时post想设置的文件内容，php执行时会将post内容当作文件内容。从而导致任意代码执行。
[GXYCTF2019]Ping Ping Ping
	==命令执行漏洞==
	使用拼接符让其执行多条命令
	“&“ ”|“ ”||“ ”&&“ ”;“
	使用ls查看文件，使用cat执行文件，绕过空格，使用$IFS变量，使用反引号\`\`高优先级执行ls,查看 源码，得到Flag() 
	关于反引号高优先级：[[【精选】Shell语法基础--各种括号、引号的用法.pdf]]
 [ACTF2020 新生赛]Exec
	==命令执行漏洞==
	index.php使用了exec来模拟执行命令，使用拼接符来执行ls,ls /,发现flag在上一级目录中，使用 cat /flag获取flag
	关于命令漏洞：[[【精选】命令执行漏洞详解_命令执行漏洞原理.pdf]]
[极客大挑战 2019]BuyFlag
	==PHP绕过==
	isnumeric()绕过:
	观察源码提示，需要绕过isnumeric()函数，可以在数值后加入空字符%00，%20或者任意字符
	弱等于(" == ")绕过:
	"admin1" == 1:False
	"1admin" == 1:True
	字符串将会被强转类型进行比较，只有数字在字符串开头时才会转为此数字，否则为0。
	而格式类似于“4e114514”会被识别为科学计数法，“0e114514”将会是0。
	绕过上面两个后，他会检测你是不是特定用户，在cookie里找到了user变量，修改其值即可。cookie也是要注意的点。
	PS:这题不用post方法不会返回FLAG，要在burpsuite进行POST提交，修改GET为POST,添加Content-Type: application/x-www-form-urlencoded，才能正常的进行POST提交。
[极客大挑战 2019]EasySQL
	==SQL注入漏洞==
	构造URLhttp://cb544b05-5897-4925-8632-8b81aace8260.node4.buuoj.cn:81/check.php?username=‘ or 1=1 -- ’&password=2
	万能钥匙(随便找的文章，后面好好看SQL注入)
	-- 为SQL注释，--后有空格(可以替换为其他符号),password被注释掉，1=1为永真条件，直接通过。
	关于SQL漏洞：[[网络安全——SQL注入漏洞.pdf]]
