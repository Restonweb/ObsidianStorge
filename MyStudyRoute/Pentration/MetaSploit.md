开源使用其控制台：msfconsole
查找漏洞-自定义漏洞-利用易受攻击的服务
step1:search
step2:use-set
step3:exploit

search:查询相关模块
use:使用模块，后可以跟查询到的序号，如use 1
set:set \[各项目标值] 切换模块值会丢失 unset清除值，unset all清除所有
setg:设置默认值 切换模块值仍为设定的默认值
这些值可以使用show options命令在正在use的模块中查看相关值的设定
show:后跟options payload等等等，显示相关信息
info:显示当前模块的信息
exploit(run):执行模块
sessions查看当前会话，与会话交互使用 sessions -i \[序号]

back退出当前的模块上下文

活用其中的auxiliary与scanner进行前期的侦察