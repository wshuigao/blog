---
title: Linux忘记mysql密码修改密码
comments: "true"
reward: "true"
categories: "坑儿~"
tags:
    - mysql
    - mysql配置
---

Linux上的mysql搭建成功以后，一段时间不用，想本地再访问mysql数据库，发现密码忘记了需要以下几步：
第一步：查询服务器上的mysql是否启动
``` bash
$ ps -ef | grep -i mysql
```

启动的话，进行关闭
``` bash
$ service mysql stop
```

第二步：  安全启动mysql，且跳过授权表： 

``` bash
$ mysqld_safe --user=mysql --skip-grant-tables --skip-networking &   # '&' 这个是也要的  不是打错了
```

上面的执行完成之后，进入 mysql  mysql -u root -p，直接回车进入就行，不需要输入密码
		
进入mysql以后  选择数据库 use mysql;  再执行进行修改密码：
``` bash
$ update mysql.user set authentication_string=password('123456') where user='root';
```
or
``` bash
$ update mysql.user set password=password('123456') where user='root';  #我的是这个生效了 
```

然后让修改立即生效：
``` bash
$ flush privileges; # 重载系统权限
```

quit 退出MySQL;

``` bash
$ service mysql restart  #重启mysql服务
```


-- -----------------------------------------
<font color="red">如果报错：can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)</font>
``` bash
$ rm -rf /var/lib/mysql #删除原来安装过的mysql残留的数据
$ systemctl restart mysqld #重启mysqld服务
```

 
			
 
   
   

