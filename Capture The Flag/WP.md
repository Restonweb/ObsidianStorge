[BUUCTF] WEB PHP1
	PHP反序列化漏洞，绕过函数 *PHP序列化与反序列化，对象属性值修改绕过*
[ACTF2020 新生赛]Include
	Include文件包含漏洞,PHP伪协议相关中的php://协议
	php://filter用于读取源码(以base64编码输出)：
	一些敏感信息会保存在php文件中，直接打开是不会将它们显示在界面上的
	但是经过base64编码再解码就可以得到其完整的源码信息
	
	php://input用于执行php代码：
	