# SQL优化
## 一. 导致索引无效的情况
#### 1. 对索引列进行运算导致索引失效,我所指的对索引列进行运算包括(+，-，*，/，! 等)
错误的例子：select * from test where id-1=9;
正确的例子：select * from test where id=10;

#### 2. 如果条件中有or，即使其中有条件带索引也不会使用
这也是为什么尽量少用or的原因，要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引

#### 3. 前导模糊查询不能利用索引(like '%XX'或者like '%XX%')
如 where code like '%AB'

由于前面是模糊的，所以不能利用索引的顺序，必须一个个去找，看是否满足条件。这样会导致全索引扫描或者全表扫描。

#### 4. 索引无法存储null值
1. 单列索引无法储null值，复合索引无法储全为null的值。
2. 查询时，采用is null条件时，不能利用到索引，只能全表扫描。

为什么索引列无法存储Null值？

1. 索引是有序的。NULL值进入索引时，无法确定其应该放在哪里。（将索引列值进行建树，其中必然涉及到诸多的比较操作，null 值是不确定值无法　　
2. 比较，无法确定null出现在索引树的叶子节点位置。）　


如果需要把空值存入索引，方法有二：

- 其一，把NULL值转为一个特定的值，在WHERE中检索时，用该特定值查找。
- 其二，建立一个复合索引

#### 5. 索引列使用表达是或函数
#### 6. not in ,not exist. 
#### 7. 范围条件后列上索引失效

```
select * from student where age > 1 and name = '王五';
```

#### 8. 使用 is null 或者 is not null 也不能使用索引

## SQL优化
#### 1. 通过变量的方式来设置参数

```
string sql = "select * from people p where p.id = ? ";
```

数据库的SQL文解析和执行计划会保存在缓存中，但是SQL文只要有变化，就得重新解析。“…where p.id = ”+id的方式在id值发生改变时需要重新解析，这会耗费时间。


#### 2. 不要使用select *
使用 
```
string sql = "select people_name, pepole_age from people ";
```

而不是

```
string sql = "select * from people "
```

使用select *的话会增加解析的时间，另外会把不需要的数据也给查询出来，数据传输也是耗费时间的，比如text类型的字段通常用来保存一些内容比较繁杂的东西，如果使用select *则会把该字段也查询出来。

#### 3. 谨慎使用模糊查询
当模糊匹配以%开头时，该列索引将失效，若不以%开头，该列索引有效。

好：

```
string sql = "select * from people p where p.id like 'parm1%' "
```

坏:

```
string sql = "select * from people p where p.id like '%parm1%' ";
```

#### 4. 对查询进行优化，要尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
#### 5. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描
最好不要给数据库留NULL，尽可能的使用 NOT NULL填充数据库. 不好的例子:

```
select id from t where num is null
```

#### 6. 应尽量避免在 where 子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描。

#### 7. limit千万级分页的时候优化。
参考: https://www.cnblogs.com/shiwenhu/p/5757250.html
https://blog.csdn.net/mqsyoung/article/details/80184700

在我们平时用limit,如：

```
Select * from trans order by id limit 1,10;
```

这样在表数据很少的时候，看不出什么性能问题，倘若到达千万级，如：

```
Select * from trans order by id limit 10000000, 10;
```

虽然都是只查询10记录，但是这个就性能就让人受不了了。意思是扫描满足条件的10000010行，扔掉前面的10000000行，返回最后的10行，问题就在这里。


**众所周知利用索引查询的语句如果只包含索引列（覆盖索引），那么它查询的速度是非常快的**。利用索引做以下优化 

我们可以用另外一种语句优化，如：

```
Select * from trans t1 join (Select id from trans order by id limit 10000000, 10) t2 on t1.id = t2.id;
```

#### 8. 应尽量避免在 where 子句中使用 or 来连接条件
如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描

```
select id from t where num=10 or num=20
```
	
可以这样查询：

```	
select id from t where num=10
union all	
select id from t where num=20
```

#### 9. in 和 not in 也要慎用，否则会导致全表扫描
```
select id from t where num in(1,2,3)
```

对于连续的数值，能用 between 就不要用 in 了：

```
select id from t where num between 1 and 3
```

#### 10. 应尽量避免在where子句中对字段进行函数或表达式操作
这将导致引擎放弃使用索引而进行全表扫描

例如: 

```
select id from t where substring(name,1,3) = ’abc’ 
```

#### 11. 索引并不是越多越好
索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率

#### 12. 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算
否则系统将可能无法正确使用索引。


#### 避免频繁创建和删除临时表，以减少系统表资源的消耗。


## 其他
#### 1. 优化选择的数据类型
Mysql支持很多不同的数据类型，选择正确的数据类型对于获得高性能至关重要。

1. 更小通常更好。更小的类型通常更快，使用更少的磁盘空间、内存和CPU缓存。比如性别使用TINYINT 类型，不用INT
2. 尽量避免null. 引用可空的查询，会使索引、索引统计和值更加复杂。要尽可能把字段定义为NOT NULL，即使应用无需保存NULL，可以用默认值设置。
3. 时间尽量使用时间类型而不是选择字符串

VARCHAR 可以节省空间，但因为长度可变所以会引起额外的工作。

#### 2. 索引支持的查询方式
1. 匹配全名：全键值匹配指和索引中的所有列匹配。
2. 匹配最左前缀
3. 匹配列前缀：可以匹配某列值的开头部分。
4. 匹配范围值： select * from test_table where age > 0 and age < 25.
5. 精确匹配一部分并且匹配某个范围中的另一部分
6. 只访问索引的的查询

#### 3. 隔离列，是索引列不是表达式的一部分或位于函数中
如果查询中没有隔离索引的列，Mysql通常不会使用索引。 隔离列即索引列不是表达式的一部分或位于函数中。

如不能查询 actor_id 的索引:

```
select actor_id from actor WHERE actor_id + 1 = 5;
```

或者date_col也不能被索引：

```
select ... where date_col >= DATE_SUB(CURRENT_DATE, INTERVAL 10 DAY)
```

#### 4. 前缀索引和索引选择性
列前缀通常可以提供高性能所需的足够选择性。如果索引BLOB和TEXT列，或者很长的VARCHAR列，就必须定义前缀索引，因为MySQL不允许索引它们的全文。


需要考虑前缀的选择，最好的前缀长度是它的选择性接近列全长的选择性。


#### 5. 聚集索引
最好避免随机(乱序)聚集键，例如使用UUID值是不好的选择：它使得聚集索引插入是随机的，这是最坏的，并且是数据聚集完全没有帮助。


# 参考
- [数据库SQL优化大总结1之- 百万级数据库优化方案](https://blog.csdn.net/wuhuagu_wuhuaguo/article/details/72875054)
- [SQL优化避免索引失效](https://blog.csdn.net/qq_23217629/article/details/52516277)
- [索引会失效的原因分析及解决索引失效的方法](https://www.2cto.com/database/201712/702834.html)


