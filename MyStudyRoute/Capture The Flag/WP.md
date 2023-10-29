[BUUCTF] WEB PHP1
#PHP反序列化漏洞
绕过函数 *PHP序列化与反序列化，对象属性值修改绕过*
扫描文件，得到源码，分析函数作用。
[ACTF2020 新生赛]Include
#Include文件包含漏洞
PHP伪协议相关中的php://协议
php://filter用于读取源码(以base64编码输出)：
一些敏感信息会保存在php文件中，直接打开是不会将它们显示在界面上的
但是经过base64编码再解码就可以得到其完整的源码信息：
php://filter/convert.base64-encode/resource=文件路径
php://input用于执行php代码：
可以访问请求的原始数据的只读流, 将post请求中的数据作为PHP代码执行。当传入的参数作为文件名打开时，可以将参数设为php://input,同时post想设置的文件内容，php执行时会将post内容当作文件内容。从而导致任意代码执行。
[GXYCTF2019]Ping Ping Ping
#命令执行漏洞
使用拼接符让其执行多条命令
“&“ ”|“ ”||“ ”&&“ ”;“
使用ls查看文件，使用cat执行文件，绕过空格，使用$IFS变量，使用反引号\`\`高优先级执行ls,查看 源码，得到Flag() 
关于反引号高优先级：[[【精选】Shell语法基础--各种括号、引号的用法.pdf]]
 [ACTF2020 新生赛]Exec
#命令执行漏洞
index.php使用了exec来模拟执行命令，使用拼接符来执行ls,ls /,发现flag在上一级目录中，使用 cat /flag获取flag
关于命令漏洞：[[【精选】命令执行漏洞详解_命令执行漏洞原理.pdf]]
[极客大挑战 2019]BuyFlag
#PHP绕过
isnumeric()绕过:
观察源码提示，需要绕过isnumeric()函数，可以在数值后加入空字符%00，%20或者任意字符
弱等于(" == ")绕过:
"admin1" == 1:False
"1admin" == 1:True
字符串将会被强转类型进行比较，只有数字在字符串开头时才会转为此数字，否则为0。
而格式类似于“4e114514”会被识别为科学计数法，“0e114514”将会是0。
绕过上面两个后，他会检测你是不是特定用户，在cookie里找到了user变量，修改其值即可。cookie也是要注意的点。
PS:这题不用post方法不会返回FLAG，要在burpsuite进行POST提交，修改GET为POST,添加Content-Type: application/x-www-form-urlencoded，才能正常的进行POST提交。
[极客大挑战 2019]EasySQL
#SQL注入漏洞
构造URLhttp://cb544b05-5897-4925-8632-8b81aace8260.node4.buuoj.cn:81/check.php?username=‘ or 1=1 -- ’&password=2
万能钥匙(随便找的文章，后面好好看SQL注入)
-- 为SQL注释，--后有空格(可以替换为其他符号),password被注释掉，1=1为永真条件，直接通过。
关于SQL漏洞：[[网络安全——SQL注入漏洞.pdf]]
[强网杯 2019]随便注
#SQL注入漏洞
加单引号报错：可注入，单引号闭合
使用Order by 命令试有几列数据
一个提交框可以搜索，其搜索的是表一的数据
使用’;’拼接多条SQL语句，使用show语句查看数据库的结构
连续使用show database;show tables;show columns from 表名;
得到其拥有两个表，在第二个表中有Flag列
但是直接进行SELECT返回被正则拦截，尝试异或字符绕过正则失败，继续使用堆叠多条SQL语句，将表一改名，将表二改为表一的名字，添加表一的id列，交换flag的位置,使用其自带的提交框进行检索id1，拿到Flag。
关于绕过正则：[[命令执行中关于PHP正则表达式的一些绕过方法.pdf]]
关于show()：[[MySQL show()函数详述.pdf]]
[SUCTF 2019]EasySQL
#SQL注入漏洞
输入1,2,3,4.......回显一个数据，输入0，无回复，输入字符无回复，堆叠语句连续使用show语句，得到表名为Flag的表，但是最后使用show columns from flag时回复nonono，其他的字符都无回显，尝试发现，flag,from被屏蔽，回复nonono
你管这叫EASY?
直接看别人WP!秒了！
猜测SQL语句SELECT ?||flag FROM Flag
法一：让语句里这个‘||’变成连接符，而不是或，使用1;set sql_mode=PIPES_AS_CONCAT;SELECT 1
法二：构造*，1
即SELECT * , 1||flag FROM Flag
关于||：
Mysql中的或运算符是通过让两边转为BOOL值再比较。
字符串为0,非0数字为1
 [极客大挑战 2019]Secret File
 #Include文件包含漏洞 
 进入页面，查看源代码，发现页面有隐藏按钮，直接跳转到Archiveroom.php,再点击到SECRET页面
 点点点直接END,没有任何的信息，让回去再看看，发现跳转用的action.php一直没有访问，直接查看其源码：![[Pasted image 20231020205329.png]]
 访问这个secr3t.php，进入后：
 ![[Pasted image 20231020205518.png]]
 其屏蔽了，../访问上一级，tp,input但是没有过滤filter，使用filter（php://filter/convert.base64-encode/resource=文件路径）查看flag.php的源码，base64解码拿到flag：![[Pasted image 20231020205753.png]]
[极客大挑战 2019]Http1
直接访问没有任何东西，利用burpsuite访问发现其下有Secret.php页面，进入后，提示，请从https://Sycsecret.buuoj.cn访问，这里要加Referer请求头，修改来源界面为https://Sycsecret.buuoj.cn.添加后访问，其提示要使用Syclover浏览器访问，将User-Agent里的Mozilla修改为Syclover即可进入，其提示需要本地访问，添加X-Forwarded-For字段：127.0.0.1，即可获取到Flag.
请求头的类型：[[http请求头有哪些？_http协议请求头_Mrwang21的博客-CSDN博客.pdf]]
[极客大挑战 2019]Knife
#命令执行漏洞 
直接访问提示：
![[Pasted image 20231023175609.png]]
那么应该是命令执行漏洞，使用PHP的eval函数来执行POST请求发送的Syc参数。
向其发送POST请求Syc=var_dump(scandir(’/‘))，其返回所在目录的结构，查看源代码(黑色背景看不到)，看到当前目录有flag文件，发送POST请求Syc=var_dump(get_file_contents('/flag'))返回flag。
![[Pasted image 20231023175937.png]]
var_dump打印变量。scandir扫描目录，返回数组。get_file_contents将文件读入字符串返回
[极客大挑战 2019]Upload
#文件上传漏洞 
上传一句话木马图片，提示非图片，加上图片头，过检测，提示包含< ？使用js执行php脚本<script language='php'>eval($_GET[cmd]);</script>,上传成功，但是其不会自动执行，在burpsuite抓包之后，修改其后缀为phtml(包含php的html)，猜测其在upload文件夹下，进入执行，通过system()请求ls命令以及ls /，发现根目录下存在flag,执行cat flag，获取到flag。
常用的文件头：[[常用文件头]]
[ACTF2020 新生赛]Upload
#文件上传漏洞 
直接用上面的🐎就可以了，步骤同上
[极客大挑战 2019]BabySQL
#SQL注入漏洞
此题依旧使用万能钥匙即可登录（Mariadb ver.'1' or 1=1#'）登陆后使用orderby找回显点,使用组合语句进行查找数据库，表，列，数据。
但是发现其屏蔽了大多数的sql语法，or from where等，使用==双写法==绕过。
连续使用以下双写过的语句。即可获取flag。
group_concat(table_name) from information_schema.tables where table_schema=database()
group_concat(column_name) from information_schema.columns where table_name=''
group_concat(table_name) frfromom infoorrmation_schema.tables whwhereere table_schema=database()
group_concat(column_name) frfromom infoorrmation_schema.columns whwhereere table_name=''
group_concat(col1,col2,col3...) from db.table
关于这些函数 ：[[内置函数与特殊数据库.pdf]]
[ACTF2020 新生赛]BackupFile
使用dirsearch扫描，设置低线程，延时，扫到index的bak备份文件：
```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}
```
将字符串视作int时，数就是字符串开头的数字。‘123’，直接key=123拿到flag
[RoarCTF 2019]Easy Calc
#命令执行漏洞 
查看源码中的script部分以及有WAF的提示：
```php
#<!--I've set up WAF to ensure security.-->
<script>
    $('#calc').submit(function(){
        $.ajax({
            url:"calc.php?num="+encodeURIComponent($("#content").val()),
            type:'GET',
            success:function(data){
                $("#result").html(`<div class="alert alert-success">
            <strong>答案:</strong>${data}
            </div>`);
            },
            error:function(){
                alert("这啥?算不来!");
            }
        })
        return false;
    })
</script>
```
注意这个encodeurl，搁burpsuite里repeater怎么send都不起作用，返回badrequest。。。要urlencode后再请求！！！
查看cacl.php:
```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>
```