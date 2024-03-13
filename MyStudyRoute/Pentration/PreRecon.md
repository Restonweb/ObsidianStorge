方向：
域名信息
	对应IP收集
		nslookup,各种网站（有可能有CDN防护）
	子域名收集
		Layer,subDomainsBrute
	Whois(注册人)信息查询
		根据已知域名反查，分析出此域名的注册人，邮箱，电话
		工具：爱站网，站长工具，微步在线
		site.ip138，searchdns.netcraft.com
敏感目录
	收集方向
		robots.txt 后台目录 安装包 上传目录 mysql管理接口 phpinfo 编辑器 iis段文件 分析网站cms
	常用工具
		字典爆破：御剑，dirbuster wwwscan IIS_shortname _Scan_
		蜘蛛爬行：爬行菜刀 webrobot burp等
		
旁站C段
	旁站：同服务器其他站点
	C段：同一网段内其他服务器
	收集方向：域名(网站)，端口(portscan)，目录(目录扫描工具)
整站分析
	服务器类型：平台，版本
	网站容器：搭建网站的服务组件：IIS，apache,nginx,tomcat
	脚本类型：ASP,ASPX,PHP,JSP等
	数据库类型：access,sqlserver,mysql,oracle,postgresql等
	CMS类型：robots文件，登陆后台，工具
	WAF：
	
GoogleHack：intext:,intitle.Filetype,inurl,site
![[Pasted image 20240313170106.png]]
URL采集
信息分析
网络设备信息收集
![[Pasted image 20240313170229.png]]
![[Pasted image 20240313170326.png]]
![[Pasted image 20240313170352.png]]
![[Pasted image 20240313170544.png]]