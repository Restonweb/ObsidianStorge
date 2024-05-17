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

 