# 常见的SQL注入
## 1.没有正确过滤转义字符
在用户的输入没有为转义字符过滤时，就会发生这种形式的注入式攻击，它会被传递给一个SQL语句。这样就会导致应用程序的终端用户对数据库上的语句实施操纵。比方说，下面的这行代码就会演示这种漏洞：

```
statement := “SELECT* FROM users WHERE name = ‘” + userName + “’;”
```

　这种代码的设计目的是将一个特定的用户从其用户表中取出，但是，如果用户名被一个恶意的用户用一种特定的方式伪造，这个语句所执行的操作可能就不仅仅是代码的作者所期望的那样了。例如，将用户名变量(即username)设置为：

```
　　a’ or ‘t’=’t
```

整个语句变成了：

```
SELECT * FROM users WHERE name = ‘a’ OR ‘t’=’t’;
```


## 除号
```
SELECT 1/0 FROM users WHERE username=’Ralph’。
```

如果WHERE语句为真，这种类型的盲目SQL注入会迫使数据库评判一个引起错误的语句，从而导致一个SQL错误。如果用户Ralph存在的话，被零除将导致错误。


## 恒等式

```
SELECT * FROM users WHERE name = ‘a’ OR ‘t’=’t’;
```



