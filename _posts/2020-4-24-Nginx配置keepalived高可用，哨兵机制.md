---
layout: post
title:  "Nginx配置keepalived高可用，哨兵机制"
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

## keepalived配置文件

```shell
! Configuration File for keepalived
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
#    vrrp_skip_check_adv_addr
#    vrrp_strict
#    vrrp_garp_interval 0
#    vrrp_gna_interval 0
}
vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh" #运行脚本，脚本内容下面有，就是起到一个nginx宕机以后，自动开启服务
    interval 2 #检测时间间隔
    weight -20 #如果条件成立的话，则权重 -20
}
# 定义虚拟路由，VI_1 为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
    state MASTER #来决定主从
    interface ens33 # 绑定虚拟 IP 的网络接口，根据自己的机器填写
    virtual_router_id 121 # 虚拟路由的 ID 号， 两个节点设置必须一样
    mcast_src_ip 192.168.206.22 #填写本机ip
    priority 100 # 节点优先级,主要比从节点优先级高
    nopreempt # 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
    advert_int 1 # 组播信息发送间隔，两个节点设置必须一样，默认 1s
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 将 track_script 块加入 instance 配置块
    track_script {
        chk_nginx #执行 Nginx 监控的服务
    }

    virtual_ipaddress {
        192.168.206.110 # 虚拟ip,也就是解决写死程序的ip怎么能切换的ip,也可扩展，用途广泛。可配置多个。
    }
}
virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
}

```

注意下图的注释部分。我的版本必须注释掉 不然报错

![image-20200425173526462](/img/image-20200425173526462.png)

## 



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

注意 关闭的时候使用kill的那个命令

```shell
1;systemctl daemon-reload  重新加载

2:systemctl enable keepalived.service  设置开机自动启动

3:systemctl disable keepalived.service 取消开机自动启动

4:systemctl start keepalived.service 启动

5:systemctl stop keepalived.service停止
systemctl status keepalived.service查看服务状态
systemctl kill keepalived 关闭
```

发现stop命令不好使的话，像下面一样注释掉killmode选项

![image-20200425184251382](/img/image-20200425184251382.png)

## /etc/keepalive下编写监控nignx脚本

```shell
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi

```

![image-20200425185000890](/img/image-20200425185000890.png)

## 如果发现并不能启动服务------->查找错误日志

**在/var/log/messages下**查看错误日志，这里发现是网卡问题

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

