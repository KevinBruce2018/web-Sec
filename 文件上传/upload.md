最近正在学习文件上传，以下是其中的一些内容。

#### php代码示例
```php
<?php
	if($_FILES['file']['error']>0)
	{
		#error返回的是错误代码
		echo "wrong code ".$_FILES['file']['error']."<br>";
		die("上传失败!");
	}
	else
	{
		echo "上传文件名: " . $_FILES["file"]["name"] . "<br>";
    	echo "文件类型: " . $_FILES["file"]["type"] . "<br>";
    	echo "文件大小: " . ($_FILES["file"]["size"] / 1024) . " kB<br>";
	    echo "文件临时存储的位置: " . $_FILES["file"]["tmp_name"];
	}
	$whitelist = array("png","gif","jpg","jpeg");
	$whitelist[] = "bmp";
	$type = $_FILES['file']['type'];
	#不想让判断语句看着太长就分开写了
	if($type=="image/png" || $type=="image/jpg" || $type=="image/jpeg" || $type=="image/gif")
	{
		$fileext = explode(".", $_FILES['file']["name"]);
		$fileext = end($fileext);
		if(in_array($fileext, $whitelist))
		{
			echo "<br>文件格式合法！<br>";
		}
		else
		{
			echo "<br>文件格式非法!<br>";
		}
	}
	else
	{
		 die("<br>文件格式非法!<br>");
	}
	if(file_exists('upload/'.$_FILES['file']['name']))
	{
		echo "文件已存在";
	}
	else
	{
        #该函数的作用是只有通过HTTP POST方式合法上传的文件才可以被保存
		move_uploaded_file($_FILES['file']['tmp_name'],'upload/'.$_FILES['file']['name']);
		echo "上传成功";
	}
?>
```
#### 注意事项
1. 如果缺少如下验证后缀的代码，可能会产生上传漏洞，黑客通过burpsite修改content-type为image/xxx，即可完成木马文件的上传。
```php
<?php
$fileext = explode(".", $_FILES['file']["name"]);
		$fileext = end($fileext);
		if(in_array($fileext, $whitelist))
		{
			echo "<br>文件格式合法！<br>";
		}
		else
		{
			echo "<br>文件格式非法!<br>";
		}
?>
```
![content-type欺骗](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/fileinclude/contenttype.png)
>访问对应的url确实上传成功了，但是该bp版本不识别中文。

2. $_FILES超级变量不一定用file数组，具体用什么参数要参考表单。比如对于下面这个，就要用$_FILES['files']。不过不管参数名叫什么，数组的几个参数都是固定的，即name,content-type,error,size,tmp_name。
```html
<!DOCTYPE html>
<html>
<head>
	<title>文件上传</title>
	<meta charset="utf-8">
</head>
<body>

<form method="post" enctype="multipart/form-data" action="upload.php">
	<input type="file" name="files" id="file"><br>
	<input type="submit" name="submit" value="提交">
</form>

</body>
</html>
```

#### python上传脚本
这里尝试一下借助python实现文件上传。
```python
#python3
import requests
url = 'http://192.168.142.132/upload.php'
#注意索引不一定叫file，根据实际情况来判断
#文件的打开方式虽然没限定，不过我看网上用rb的居多
files = {"file":("shell.php",open('shell.php'),"image/png")}
r = requests.post(url,files=files)
r.encoding = r.apparent_encoding
print(r.text)
```
#### 上传绕过
虽然加了后缀的限制，但是也不是不能绕过去。（当然，有一些骚的，根据你的contype-type替你把后缀改了，这个很恶心人了）

apache和IIS均具有一些解析漏洞可以利用。对于apache（版本暂时不清楚），他会以第一个.后面的后缀名作为文件后缀进行解析。比如a.php.jpg,会被当做PHP进行解析。

对于IIS，IIS6.0比较容易，如果存在a.asp目录，则该目录下无论是什么后缀都会被当做asp进行解析。

对于IIS7.0 7.5，会对分号产生截断。比如a.asp;1.jpg,在解析的时候会被解析成a.asp。在cgi.fixphpinfo开启的情况下，上传的PHP文件会由IIS交给PHP解析，否则不会。

>常见的截断方式有分号截断，%00截断（a.jpg%00.php），但是PHP5.3.4以后就不支持00截断了。

除了检查后缀，也可能检查内容，这种情况下就需要免杀马。