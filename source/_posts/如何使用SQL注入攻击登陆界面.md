---
title: 如何用SQL注入攻击登陆界面
date: 2017-08-30 17:37:08
tags: [安全, SQL攻击]
---

### 适用范围：
1. 如果一个系统是通过

```bash
SELECT * FROM accounts WHERE username='admin' and password = 'password'
```

这种显式的SQL来进行登陆校验，也就是执行这个SQL语句，如果数据库中存在用户名为admin，password的用户，就登陆成功，否则就登陆失败。

2. 系统没有对用户输入进行全面的过滤。

3. 系统后台使用的是MYSQL数据库。

4. 系统存在一个username为admin的用户

### 攻击原理

利用MYSQL的注释功能，也就是"/\*"，MYSQL执行SQL脚本时，如果遇到/\*标示符，就会把之后的SQL当做注释而不会执行。

正常情况下用户在用户名框内输入"admin"，在password框内输入"password"，后台执行SQL语句为
```bash
SELECT * FROM accounts WHERE username='admin' and password = 'password'
```

但是如果用户名框内输入`admin' AND 1=1 /\*`，在密码框内输入任意字符串，那么后台执行的SQL就为
```bash
SELECT * FROM accounts WHERE username='admin' AND 1=1 /* and password='aa'
```
可以看到数据库实际执行的SQL为
```bash
SELECT * FROM accounts WHERE username='admin' AND 1=1
```
而/\*后面的SQL就被当做注释而忽略掉了，登陆成功！
