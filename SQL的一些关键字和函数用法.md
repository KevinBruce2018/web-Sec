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

5.limit和offset

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

6.concat()

可以将多个字符串连接起来，参数个数无限。

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

7.updatexml()

这个还不太会用，先抛出一个例子，主要是利用其报错把查询的结果给显示出来。

如果关了回显的话就不要用这个了。

select updatexml(1,concat(0x7e,version(),0x7e),1);

第二个参数由于不符合xpath的规范，会报错，但是报错的时候会把整个字符串的结果报出来，因为假设查询了flag，错误回显中会出现flag。
