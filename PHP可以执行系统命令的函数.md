### PHP中可以执行系统命令的函数
>https://blog.csdn.net/Auuuuuuuu/article/details/88778852
#### system()
函数原型:string system ( string $command [, int &$return_var ] )

该函数执行会有回显，执行的结果会回显到页面上。可选参数return_var会返回一个执行后的状态码。
```php
<?php
    system("whoami");
?>
```

#### exec()
函数原型:string exec ( string $command [, array &$output [, int &$return_var ]] )

该函数执行后无回显，默认返回最后一行的结果。

函数执行后的全部结果放在$output变量中，该参数在使用时是可选的。

```php
#只会返回dir结果的最后一行
<?php
$arr = exec("dir");
echo "$arr";

#返回所有结果并打印
$arr = exec("dir",$output);
for($i = 0;$i < count($output);$i++)
    echo $output[$i];
?>
```

#### shell_exec()
函数原型:string shell_exec( string &command)

该函数无回显，但是会将执行后得到的内容作为返回值返回。
```php
<?php echo shell_exec("dir"); ?>
```
该函数实际上是反引号的变体，当禁用shell_exec时，` 也不可执行。上面的代码与下面的代码是等价的。
```php
<?php echo `dir`; ?>
```
<b>PHP会尝试将反引号中的字符串作为系统命令进行执行。</b>

#### passthru()
函数原型:void passthru ( string $command [, int &$return_var ] )

用法和system没什么区别。
```php
<?php passthru("whoami"); ?>
```

#### popen()
函数原型:resource popen ( string $command , string $mode )

参数简介:
$mode是指文件的打开方式,r或w。

该函数执行后不返回执行结果，返回一个文件指针。可以借助该指针进行一些文件操作。

popen()打开一个指向进程的管道，且必须用pclose()关闭。

```php 
<?pho
//执行命令并返回一个文件指针
$fp = popen("whoami","r"); 
//读取内容
while (!feof($fp)) {
 $out = fgets($fp, 4096);  
 echo  $out;  
}  
pclose($fp);  
?>
```

#### 未完待续