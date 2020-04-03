1.database()

该函数可以显示当前正在使用的数据库库名。

2.mid()

该函数可以从指定的字段中提取出字段的内容。

mid(column_name,start[,length])
```sql
select mid(name,1) from user;
/*start要求最小从1开始!!!*/
/*从password这一列的每一个元素的第一个字符开始截取*/
/*注意得到的是整整一列的内容*/
```
举例:

```mysql
select * from user;
```

| name | password |
| ------ | ------ |
| admin | admin |
| root | root |

```mysql
select mid(name,1,2);
```

|name|
|---|
|ad|
|ro|

3.substr()

这个函数很常用，有三个参数，按顺序分别是字符串，起始位置和长度。可以求指定字符串的子串。当然，第一个参数可以是列的名字。这个函数似乎和mid没有什么不同，如果mid或者substr中的某一个函数被禁了就用另一个。

4.ord()

该函数用于获得某个字符串最开始的字符的ASCII值。

5.ascii()

目前未发现与ord的不同。不过这样也有很大好处，那就是，如果SQL注入的题目中过滤了or，ord函数会躺枪，可以用ascii函数替代。

6.limit和offset

limit和offset均用于限制查询结果显示的行数。

limit后面可以有1个或两个参数。

给出一个mysql库user表的某一列:

|host|
|---|
|host|
|host|
|localhost|
|localhost|

一个参数时
```sql
select host from user limit 2;
/*limit 0会得到空集合
limit大于查询结果返回的行数时，显示全部结果
limit负值会报错
*/
```

|host|
|---|
|host|
|host|

两个参数时，第一个参数表示开始位置（0作为最开始的位置），第二个参数表示显示的行数。

```sql
select host from user limit 1,2;
/*当参数一过大时，得到空集合
参数二过大时，从参数一的位置开始显示剩余的全部结果
*/
```

|host|
|---|
|host|
|localhost|

offset必须与limit结合着来用。

```sql
select host from user limit 1 offset 2;
/*表示从第二个开始，显示一条数据*/
/*
limit后面的参数总是限定显示多少条，明白这一点就不会错了
*/
```

7.concat()

可以将多个字符串连接起来，参数个数无限。但是需要注意的地方就是，如果有一个参数的值是NULL，那么整行结果就会返回一个NULL。

举个例子
```sql
select concat(user,' ',host)contest from user where host='localhost';
```

| contest             |
|---------------------|
| ftp localhost       |
| geez localhost      |
| mysql.sys localhost |
| root localhost      |
| stone localhost     |
| temp localhost      |

```mysql
select concat('aaa','kkk',NULL);
```

结果是NULL。

8.group_concat()

该函数可以将查询结果连成一行，如果只查询一列，默认用逗号分隔；如果查询多列，每一行的查询结果会直接进行字符串连接，行之间默认用逗号分隔。需要注意的是，用于分隔的默认字符可以修改。

举个例子：

```mysql
select group_concat(username,password)result from users;
```

|result               |
|---------------------|
|admin21232f297a57a5a743894a0e4a801fc3,testc4ca4238a0b923820dcc509a6f75849b|

9.updatexml()

这个主要是填写错误的xpath参数使查询报错，报错时会把xpath位置的查询结果暴露出来。

如果关了回显的话就不要用这个了，可以考虑时间注入。

用法举例

```mysql
select updatexml(1,concat(0x7e,version(),0x7e),1);
```

第二个参数由于不符合xpath的规范，会报错。报错的时候会把version()执行的结果报出来，假设查询了flag，错误回显中会出现flag。

```txt
ERROR 1105 (HY000): XPATH syntax error: '~5.7.17~'
```

10.left()

该函数是一个字符串处理函数，用法实例:

```mysql
select left('2019',2);
```

返回的结果为：20。

11.right()

这个和left对比着记，用法实例：

```mysql
select right('2019',2)
```

返回的结果为：19。

很明显，substr完全可以取代left和right两个函数，但是如果substr和mid被禁了，left和right就可以结合着用。

12.elt()函数

elt(n,str1,str2,str3);

该函数的作用是，返回参数中的第n个字符串,参数可以是字符串常量或者列名。

比如：

```mysql
select elt(2，'888','666');
```

其返回的结果是666。

如果第一个参数是0，则返回NULL。并且如果第一个参数是0，后面无论是什么，都不会考虑了，如果是函数，则不会运行了。

13.sleep()

该函数的参数是一个整数t，可以在执行某操作后延迟t秒而不进行任何操作。该函数常用于处理没有回显的SQL注入，根据响应的时间来确定被注入的SQL语句是否执行成功了。

14.length()

该函数的参数可以是字符串，或者列名。该函数的作用是获取字符串的长度。比如：

```mysql
select length('test');
```

得到的结果是4。

15.rand()

该函数用于产生一个随机数，可以接受一个参数作为种子，也可以直接

```mysql
select rand();
```

16.floor()

该函数用于将一个浮点数向下取整得到整数，可以与rand函数配合使用。在特定情况下，rand、floor、count(*)配合group by可以进行报错注入。

>关于rand、floor、count(*)和group by进行报错注入的方法几原理，可以参考以下文章：https://www.2cto.com/article/201604/498394.html