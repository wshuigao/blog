---
title: 使用nginx方式实现http转换为https
comments: "true"
reward: "true"
categories: "坑儿~"
tags:
    - nginx
    - 域名配置
---

   使自己的域名从http转变为https,提高网站安全性，可以防止我们的网址被劫持。
同事说很简单的 然后我就百度着手开始弄，但是我的并没他说的那么简单，简直就是一步一个错误！

   首先 把ssl证书下载下来 我这里选择的Nginx的证书
此处应有图片：。。。
   然后在服务器nginx文件夹下面 创建一个cert文件夹 把下载下来的文件上传到这个文件下面 可以用命令上传，也可以用XFtp上传

   执行命令：<b> vim nginx.conf</b> 进入修改Nginx的配置; 按i 进行编辑 ;拉到最下面 修改配置 (这是我修改好的):
~~~html
``` bash
      server {
       listen 443; //https访问的端口
       server_name www.iswho.site; //我的域名
       ssl on;
       root html/public; //nginx 静态访问的index页面
       index index.html index.htm;
       ssl_certificate   ../cert/iswho.pem; //你所上传的文件
       ssl_certificate_key  ../cert/iswho.key; //你所上传的文件
       ssl_session_timeout 5m;
       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
       ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
       ssl_prefer_server_ciphers on;
       location / {
           root html/public;
           index index.html index.htm;
       }
      }
```
~~~
修改完 按 <kbd>ESC</kbd> 退出编辑；<b> :wq</b>进行保存退出；
执行检查配置文件Nginx.conf是否正确 命令：

``` bash
$ /usr/local/nginx/sbin/nginx -t 
```
然后我这里开始报错 

``` bash
$ unknown directive "ssl" in /usr/local/nginx/conf/nginx.conf:100
```
或者是
``` bash
$ nginx:[emerg]unknown directive ssl
```
这个错误说的并不是 刚才修改的配置文件 修改错了，而是说 你的Nginx 不支持SSL，
因为我们配置这个SSL证书需要引用到Nginx的中SSL这模块，然而我们一开始编译的Nginx的时候并没有把SSL模块一起编译进去，
所以导致这个错误的出现。

可以使用 
``` bash
$ /usr/local/nginx/sbin/nginx -V 
```
命令检查 Nginx 是否支持 SSL,它会输出版本号跟是否支持ssl。
我的刚开始是这样的：configure arguments:
：后面是空的 表示不支持,需要进行配置 使其支持。

第一步：我们先来到当初下载Nginx的包压缩的解压目录
<font color="red">命令1</font>： ./configure --with-http_ssl_module  //重新添加这个ssl模块

这个时候 我又报了个错误：
``` bash
$ ./configure: No such file or directory 
```
意思是说 我没有这个文件; 也有可能是:(./configure：错误：SSL模块需要OpenSSL库。)这个错误
原因是因为缺少了OpenSSL，这2个错误解决都可以执行：
``` bash
$ yum -y install openssl openssl-devel 
```
进行 openssl的安装；
再执行 ./configure 命令(这个好像不执行也可以，我好像是没执行，但是看百度别人的有执行)
然后 重新执行命令1
<font color="red">命令2</font>：执行 make 命令，但是不要执行make install，因为make是用来编译的，而make install是安装，不然你整个Nginx会重新覆盖的。
<font color="red">命令3</font>：在我们执行完做命令后，我们可以查看到在nginx解压目录下，objs文件夹中多了一个nginx的文件，这个就是新版本的程序了。
首先我们把之前的nginx先备份一下，然后把新的程序复制过去覆盖之前的即可。
<font color="green">注：注意路径</font>

  cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak   // 拷贝原始nginx 为一个nginx.bak的副本
    cp /opt/nginx-1.11.6/objs/nginx /usr/local/nginx/sbin/nginx      // 把新生成的文件覆盖原始文件
     cp:拷贝 ；/opt/nginx-1.11.6/objs/nginx 这个是上面 执行完make命令产生的 ；/usr/local/nginx/sbin/nginx 这个是你xxx的地方
    按y 确认覆盖
    
   我记得 这个时候 好像报了一个 Text file busy 的错误
   是因为我的Nginx 已开启 正在使用中的原因 先把Nginx关闭就行了
   这时再查看 目前的nginx是否支持ssl  /usr/local/nginx/sbin/nginx -V

   ``` bash
     nginx version: nginx/1.11.6
     built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
     built with OpenSSL 1.0.2k-fips  26 Jan 2017
     TLS SNI support enabled
     configure arguments: --with-http_ssl_module 
   ```
configure arguments 这个后面有东西了 就代表支持SSL
    然后接着往下 再执行命令 检查nginx.conf的配置文件是否正确 报了一个：
    ``` bash
      nginx: [emerg] BIO_new_file("/usr/local/nginx/conf/cert/iswho.pem") failed
     (SSL: error:02001002:system library:fopen:No such file or 	  directory:fopen('/usr/local/nginx/conf/cert/iswho.pem','r')
      error:2006D080:BIO routines:BIO_new_file:no such file)
      nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
    ```
根据返回的错误可知是那个路径没对上 它找到的是conf下的cert文件夹，而我的cert文件夹再nginx文件夹下面
然后 我打开nginx.conf 重新配置了一下

``` bash
  ssl_certificate   ../cert/iswho.pem;
  ssl_certificate_key  ../cert/iswho.key; 
```
定位到了 正确的位置
保存 ，重新用命令检查 
``` bash
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```
这样是正确的

然后我重新加载一下配置 /usr/local/nginx/sbin/nginx -s reload
又来了一个错误：
``` bash
$ nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
```
执行这个命令：/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf 就ok了 
然后 vim nginx.conf 
加一句话：
server {
listen 80; //端口号
server_name www.域名.com; //域名
<font color="red">rewrite ^(.*) https://$server_name$1 permanent;</font> //重定向到https
}
用命令重新加载nginx 配置
用命令重启 nginx服务器
然后 再访问自己的域名 你会发现 域名地址前面有把锁,点击锁 提示链接是安全的，证书是有效的
或者 复制域名发给朋友  会发现域名前缀由本来的http改为了https

 <font color="red">以下包含了 Nginx 常用的几个命令：</font>

``` bash
/usr/local/nginx/sbin/nginx -t                #检查配置文件nginx.conf的正确性命令
/usr/local/nginx/sbin/nginx -s reload        # 重新载入配置文件
/usr/local/nginx/sbin/nginx -s reopen       # 重启 Nginx
/usr/local/sbin/nginx -s stop              # 停止 Nginx
```

