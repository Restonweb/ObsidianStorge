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
	使用ls查看文件，使用cat执行文件，绕过空格，使用$IFS变量，使用反引号\`\`高优先级执行ls,查看 源码，得到Flag