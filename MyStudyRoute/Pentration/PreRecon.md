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
GoogleHack
URL采集
信息分析