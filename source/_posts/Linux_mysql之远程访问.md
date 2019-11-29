---
title: windows连接linux上的mysql数据库
comments: "true"
reward: "true"
categories: "坑儿~"
tags:
    - mysql
    - mysql配置
---

Linux上的mysql搭建成功以后，想本地windows访问mysql数据库 需要以下几步：
   
第一步：给服务器配置 3306的安全组:
	比如我的是阿里云服务器，登录阿里云控制台，找到云服务器，点进去找到 实例，里面会显示一个 你的服务器，点击更多，有个网络与安全组--->安全组配置--->配置规则
	--->添加安全组--->端口号是3306--->授权对象是 0.0.0.0/0--->描述可以自定义 我的是mysql,添加完 就OK了
	
第二步：登录服务器，给mysql配置远程登录权限:
	登录自己的服务器，然后再登录mysql 进入mysql命令：
``` bash
$ mysql -u root -p mysql  # 第1个mysql是执行命令，第2个mysql是系统数据名称
```

这时有密码输入密码，没有密码直接回车 。 忘记mysql密码的可以看另外一章文章，进行修改密码
进入mysql以后 在mysql执行：
 ``` bash
$ grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option; 
# root是用户名，%代表任意主机，'123456'指定的登录密码（这个和本地的root密码可以设置不同的，互不影响）代表任何主机都可以访问
```

然后让修改立即生效，执行以下命令：
	
``` bash
$ flush privileges; # 重载系统权限
```
		
如果想允许用户root从某个固定的ip主机连接到MySQL服务,比如：192.168.137.99这个ip,执行以下命令
 ``` bash
$ grant all privileges on *.* to 'root'@'192.168.137.99' identified by '123456' with grant option;
$ flush privileges; # 重载系统权限
```

第三步：把3306端口 加到防火墙中，使其可以对外访问
	
先执行 
``` bash
$ iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```

再执行 
``` bash
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

重启防火墙
``` bash
$ systemctl restart firewalld.service
```
查看规则是否生效
``` bash
$ iptables -L -n
```
现在去windows上用navicat访问Linux的上的mysql 就可以连接了

   
 
   
   

