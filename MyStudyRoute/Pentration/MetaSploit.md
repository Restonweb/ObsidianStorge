开源使用其控制台：msfconsole
查找漏洞-自定义漏洞-利用易受攻击的服务
step1:search
step2:use-set
step3:exploit
search:查询相关模块
use:使用模块，后可以跟查询到的序号，如use 1
set:set \[各项目标值] 切换模块值会丢失
setg:设置默认值 切换模块值仍为设定的默认值
这些值可以使用show options命令在正在use的模块中查看相关值的设定
info:显示当前模块的信息
exploit(run):执行模块

back退出当前的模块