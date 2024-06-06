ASPX的环境一般可能为：
windows+iis+aspx+sqlserver
# .NET项目/ASPX
## DLL反编译
工具：ILSpy
aspx中的页面代码可能很少，形如：
![[Pasted image 20240517172452.png]]
其实现功能的函数可能被封装在DLL中
上图就指向加载了HdhCms.dll中HdhCms.Admin下类Activity的代码。且指示了源码为C#。 
![[Pasted image 20240517172659.png]]
通过分析web源码可以发现漏洞，而aspx.NET的源码如果加载DLL可以反编译分析。
## 配置信息泄露
web.config存放了网站的配置，其中可能泄露数据库的连接密码，网站类型等
其中重要的配置：
CustomError:OFF/ON
OFF:给出错误的原本格式，stackTrace等，可能会爆出一些关键的路径，源码等
ON:自定义错误，不会显示原本错误格式
## 未授权访问
未授权访问分为两种情况：
1.验证代码文件可不可以绕过
2.找有哪些文件没有包含验证代码的文件
一般带有登录授权的都会通过cookie或session来判断用户有无授权，来决定是否可以访问特定页面，如下源码(purchase.master )：
![[Pasted image 20240517181040.png]]
这样的直接抓包修改cookie——这属于第一种情况
第二种情况：
白盒情况下：
同一个大功能(purchase)下：
如果小功能页面包含了带验证代码的文件(如purchase.master)：
![[Pasted image 20240517181951.png]]
直接访问是会进行用户授权的验证的
如果小功能页面(cat_move.asapx)未包含带验证代码的文件：
![[Pasted image 20240517182021.png]]
![[Pasted image 20240517182332.png]]
那么就可以直接访问，这是不应该的，这就属于未授权访问。
黑盒情况下：
扫目录发现cat_move.aspx并没有redirect，说明存在未授权访问。 