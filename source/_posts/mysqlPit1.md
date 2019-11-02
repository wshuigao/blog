---
title: mysql数据库报错
comments: "true"
reward: "true"
categories: "坑儿~"
tags:
    - mysql
---
来到公司像往常一样打开项目启动，谁知道没启动起来,看错误日志 报了一个mysql连接不上去,包括navicat 也连接不上去 报错代码如下：

## 报错代码
``` bash
$ Lost connection to MySQL server at ‘reading initial communication packet', system error: 0
```
意思是mysql远程连接丢失

### 解决方法

windows下的mysql错误解决办法： 找到mysql文件夹的所在位置， 与mysql-->bin 目录同级的 my.ini 进行编辑
 打开以后   如果里面有 bind-address = 127.0.0.1  这句代码可以注释掉  前面加#号就可以注释掉了
 然后再 [mysqld] 的下面 添加 一行代码  
``` bash
skip-name-resolve
```
保存 重启mysql服务就可以了

> linux 下的mysql 操作基本一样 不过linux下不是my.ini文件  而是my.cnf文件
可以用 命令修改 vim my.cnf ， 打开后 输入i 进行编辑， 里面的修改 跟windows下面的操作一样
编辑完 以后 用:wq 进行保存  然后再重启mysql 


### 测试

重启完毕 把navicat 关掉 重新打开 就可以进入数据库


