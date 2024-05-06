选择题  20分
小题5道50分
`小综合  20分`
大综合  30分

# 5小题
## 1.sql注入（尽量用sqlmap）
### 可能性1：nivs二次注入
扫目录或看源码得到register注册页，注册账号，进入
管理页面提示无权限，联系leader\@hxxxx.com
改leader密码：
进入注册页面，输入上面的leader邮箱，burp开启抓包，在burp里将邮箱改为leader@hxxxx'#\@qq.com自己定一个密码。
登录leader:
进入登陆页面，因为上面注册的邮箱带单引号，仍然抓包进行修改。
