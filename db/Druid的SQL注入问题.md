# Druid的SQL注入问题
## 符号问题
druid 中的wall 会拦截非法sql, 如果在sql的 order by中没有指定 ASC， DESC就会报:

```
sql injection violation
```
比如：

```
SELECT F_user_name , F_age, F_both_day  FROM users WHERE username=’Ralph’  order by  F_both_day, F_age
```

虽然默认走的是升序，但是druid 检查到 F_both_day、F_age 没有指定排序规则，就会报错。

## 恒等式
```
SELECT * FROM users WHERE name = ‘a’ OR ‘t’=’t’;
```


