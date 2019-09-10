#### 审核完的文件
admini/login.php 无漏洞利用点

#### 一些参考链接
https://blog.csdn.net/qq_23936389/article/details/81256154
https://www.0dayhack.com/post-560.html

#### 可能有问题的地方

##### admini/controllers/system/bakup.php

安全的函数
delete()

```php
#如果被利用可以产生任意文件下载漏洞
function download()
{
	global $request;
	if(!empty($request['filename']))
	{
		file_down(ABSPATH.'/temp/data/'.$request['filename']);
	}
	else
	{
		 echo '<script>alert("文件名不能为空!");window.history.go(-1);</script>';
	}
}
//http://127.0.0.1/admini/index.php?m=system&s=bakup&a=download&filename=../../config/doc-config-cn.php&id=81256154&token=2ca717f821e8f179b22130cc625c3921'
```

```php
#可能产生任意文件读取  但是考虑到读取的是缓存  可能暂时没啥价值
function cache_read($file, $mode = 'i')
{
	$cachefile = ABSPATH.'/temp/data/'.$file;
	@chmod($cachefile, 0666);
	if(!is_file($cachefile)) return array();
	return $mode == 'i' ? include $cachefile : file_get_contents($cachefile);
}
#调用的话可以删除任意文件
function cache_delete($file)
{
	return @unlink(ABSPATH.'/temp/data/'.$file);
}
```

```php

```

```php
```