---
layout: post
title:  "Nginx配置keppalived高可用，哨兵机制"
date:   2020-4-24
categories: 高可用
tags:  Nginx
author: Clear Li
---

### 

记录一下配置时遇到的问题













## 安装nginx

这个之前操作过了，没有截图就只写步骤吧。

```shell
  $ cd /usr/local/  $ wget http://nginx.org/download/nginx-1.8.0.tar.gz  

$ tar -zxvf nginx-1.8.0.tar.gz  $ cd nginx-1.8.0  

 $ ./configure --prefix=/usr/local/nginx

 $ make  

$ make install  
```



## nginx常用命令

```shell
/usr/local/nginx/sbin/nginx 启动命令
重启：
$ /usr/local/nginx/sbin/nginx –s reload
停止：
$ /usr/local/nginx/sbin/nginx –s stop
测试配置文件是否正常：
$ /usr/local/nginx/sbin/nginx –t 
强制关闭：
$ pkill nginx

```

## 安装Keepalived(版本2.0.18)

**解压**

```shell
tar -zxvf keepalived-2.0.18.tar.gz -C /usr/local/
```

![image-20200424230409398](/img/image-20200424230409398.png)

**安装openssl-devel**

```
yum install -y openssl openssl-devel
```

![image-20200424230809657](/img/image-20200424230809657.png)

### 编译安装

```
 ./configure --prefix=/usr/local/keepalived
```

![image-20200424231103470](/img/image-20200424231103470.png)

### **make**

```
make
```

![image-20200424231429920](/img/image-20200424231429920.png)

**make install** 

```shell
make install 
```

![image-20200424231520871](/img/image-20200424231520871.png)

## 配置keppalived服务（开机自启动）



创建文件夹，将配置文件复制到文件夹中

```shell
mkdir /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/

```

**没有找到/rc.d/init.d文件夹，要在**源码文件中找

![image-20200425094933351](/img/image-20200425094933351.png)

然后复制keepalived脚本文件：(注意init文件位置)

```shell
cp /usr/local/keepalived/etc/init.d/keepalived /etc/init.d/
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
ln -s /usr/local/sbin/keepalived /usr/sbin/
ln -s /usr/local/keepalived/sbin/keepalived /sbin/

```

然后使用下面命令设置开机自启动

## systemctl常用配置

```shell
1;systemctl daemon-reload  重新加载

2:systemctl enable keepalived.service  设置开机自动启动

3:systemctl disable keepalived.service 取消开机自动启动

4:systemctl start keepalived.service 启动

5:systemctl stop keepalived.service停止
systemctl status keepalived.service查看服务状态
```



![image-20200424232133650](/img/image-20200424232133650.png)



## 发现并不能启动服务------->查找错误日志

**在/var/log/messages下**查看错误日志，发现是网卡问题

1.编写本地配置文件

![image-20200425000219905](/img/image-20200425000219905.png)

2.修改本地配置文件的网卡，改为自己网卡，这里我的网卡是eth33

![image-20200425000209005](/img/image-20200425000209005.png)

![image-20200425000354273](/img/image-20200425000354273.png)

3.etc下的配置文件的网卡也要改动。如下图所示

![image-20200425001639717](/img/image-20200425001639717.png)

## 最终效果

启动成功

![image-20200425001851453](/img/image-20200425001851453.png)