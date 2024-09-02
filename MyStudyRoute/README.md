#基于Flask实现后台权限管理系统

基于Python的Flask WEB框架实现后台权限管理系统，内容包含：用户管理、角色管理、资源管理和机构管理。前端页面参考了@sypro。

**系统已经切换python 3，我的是在python 3.7.0下测试的，理论上Python 3版本应该都是可以运行的。需要Python 2版本的朋友可以checkout到python2分支。**


**完整设计文档**
[参考百度阅读 -- 已经不可用](https://yuedu.baidu.com/ebook/8e8853732e60ddccda38376baf1ffc4fff47e278)

如果有需要详细设计电子书的同学，10元一本，一杯奶茶不到的价格，写作不容易。

加我微信支付，备注authbase。

微信号是jeffrey-chu

![机构管理](doc/wx.jpg)

**Docker运行**

我已经将系统打包到docker镜像中，镜像中包含：
1. ubuntu 20.04
2. authbase代码
3. mysql server 8.0。数据库账户密码authbase/123456


具体方法参考源码中的docker目录。

如何使用

1. docker pull docker push zisokal/authbase:1.0
2. docker run -d -p 5000:5000 \
	-e DEV_DATABASE_URI=mysql+mysqlconnector://authbase:123456@127.0.0.1/authbase?charset=utf8 \
	--name authbase authbase:1.0
3. 打开浏览器访问页面 http://localhost:5000。系统默认的登录名密码为admin/123456



**前端依赖插件**

 1. My97DatePicker
 2. jQuery
 3. Bootstrap
 4. jQuery EasyUI
 5. jQuery Portal
 

**后端依赖插件**

 1. Flask
 2. Flask-Migrate
 3. Flask-Script
 4. Flask-SQLAlchemy
 5. Flask-Login
 6. itsdangerous
 7. Jinja2
 8. Werkzeug
 9. mysql-python

**使用方法**

1. 导入根目录下db.sql数据库脚本到mysql数据库
2. pip3 install -r requirements.txt
3. python3 manager.py runserver
 
**讨论群**

欢迎加入python技术爱好者，群号码：297690915，内有福利！

**效果图**

![机构管理](doc/机构管理.png)

![角色管理](doc/角色管理.png)

![用户管理](doc/用户管理.png)

![资源管理](doc/资源管理.png)

**图书资源推荐**

![小程序](doc/扫码_搜索联合传播样式-标准色版.png)