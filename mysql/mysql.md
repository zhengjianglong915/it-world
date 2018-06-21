# Mysql
## 工具
### 时间差计算
语法：

```
TIMESTAMPDIFF(interval,datetime_expr1,datetime_expr2)
```

其结果的单位由interval 参数给出。该参数必须是以下值的其中一个：

 - FRAC_SECOND。表示间隔是微秒. 1000000 * FRAC_SECOND = SECOND
 - SECOND。秒
 - MINUTE。分钟
 - HOUR。小时
 - DAY。天
 - WEEK。星期
 - MONTH。月
 - QUARTER。季度
 - YEAR。年

### 表大小统计
select sum(DATA_LENGTH)+sum(INDEX_LENGTH) from information_schema.tables 
where TABLE_NAME='数据库名';


