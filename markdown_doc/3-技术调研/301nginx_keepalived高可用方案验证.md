---
title: 301nginx_keepalived高可用方案验证
tags: nginx-3
renderNumberedHeading: true
grammar_cjkRuby: true
---

[toc]
## 前期调研
### 本地验证
#### 安装nginx
>$docker pull nginx:latest
>$docker run --name nginx-test -p 8080:80 -d nginx
>
- --name nginx-test：容器名称。
- -p 8080:80： 端口进行映射，将本地 8080 端口映射到容器内部的 80 端口。
- -d nginx： 设置容器在在后台一直运行。


#### 安装配置keepalived
##### keepalived运行原理
Keepalived 是一个基于VRRP协议来实现的LVS服务高可用方案，可以利用其来避免单点故障。一个LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个虚拟IP，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。

在路由器上配上静态路由就会产生单点故障,那该怎么办呢?VRRP就应用而生了,VRRP通过一竞选(election)协议来动态的将路由任务交给LAN中虚拟路由器中的某台VRRP路由器.

VRRP工作原理, 在一个VRRP虚拟路由器中，有多台物理的VRRP路由器，但是这多台的物理的机器并不能同时工作，而是由一台称为MASTER的负责路由工作，其它的都是BACKUP，MASTER并非一成不变，VRRP让每个VRRP路由器参与竞选，最终获胜的就是MASTER。MASTER拥有一些特权，比如，拥有虚拟路由器的IP地址，我们的主机就是用这个IP地址作为静态路由的。拥有特权的MASTER要负责转发发送给网关地址的包和响应ARP请求。

VRRP通过竞选协议来实现虚拟路由器的功能，所有的协议报文都是通过IP多播(multicast)包(多播地址224.0.0.18)形式发送的。虚拟路由器由VRID(范围0-255)和一组IP地址组成，对外表现为一个周知的MAC地址。所以，在一个虚拟路由 器中，不管谁是MASTER，对外都是相同的MAC和IP(称之为VIP)。客户端主机并不需要因为MASTER的改变而修改自己的路由配置，对客户端来说，这种主从的切换是透明的。

在一个虚拟路由器中，只有作为MASTER的VRRP路由器会一直发送VRRP通告信息(VRRPAdvertisement message)，BACKUP不会抢占MASTER，除非它的优先级(priority)更高。当MASTER不可用时(BACKUP收不到通告信息)， 多台BACKUP中优先级最高的这台会被抢占为MASTER。这种抢占是非常快速的(<1s)，以保证服务的连续性。由于安全性考虑，VRRP包使用了加密协议进行加密

##### 安装最新版本
>$cd /usr/local/src/
$wget http://www.keepalived.org/software/keepalived-2.0.19.tar.gz
$tar -zxvf keepalived-2.0.19.tar.gz
$cd keepalived-2.0.19
$./configure --prefix=/usr/local/keepalived 
$make && make install

##### 配置
查看keepalived目录层级结构
>$tree -l /usr/local/keepalived/etc

keepalived启动时会从服务器的/etc/keepalivec目录下查找keepaplived.conf配置文件，需要手动创建目录，将文件拷贝到目录下
>$ mkdir /etc/keepalived
>$ cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
>$ cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/rc.d/init.d/keepalived(这行执行不了，目录下无该文件,1.4版本后就没有,修改命令为下面两行。)  
> $ cp /usr/local/src/keepalived-2.0.19/keepalived/etc/init.d/keepalived /etc/init.d
> $ mkdir /etc/sysconfig(新增)
> $ cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/keepalived 
>$ ln -s /usr/local/sbin/keepalived /usr/sbin/
>$ ln -s /usr/local/keepalived/sbin/keepalived /sbin/


设置keepalived服务开机启动
> $chkconfig keepalived on (找不到命令)

添加权限
> $chmod 755 /etc/init.d/keepalived

启动
>$ service keepalived start   #启动服务
$ service keepalived stop    #停止服务
$ service keepalived restart #重启服务
```
$ sudo apt-get install keepalived

[root@localhost ~]# systemctl start keepalived   //启动keepalived
[root@localhost ~]# systemctl enable keepalived  //加入开机启动keepalived
[root@localhost ~]# systemctl restart keepalived  //重新启动keepalived
[root@localhost ~]# systemctl status keepalived   //查看keepalived状态

ps -ef | grep keepalives
```


#### 两个例子keepalived配置选项全说明

```
! Configuration File for keepalived
global_defs {
	notification_email  #keep
alived切换时需要发送email地址，也可以通过其他方式报警，非实时通知
		zhouxiao@example.com
		itsection@example.com
	}
	notification_email_from itsection@example.com
	smtp_server mail.example.com
	smtp_connect_timeout 30
	router_id LVS_DEVEL #机器标识，通常设置为hostname。故障发生时，邮件会通知到
}
vrrp_script chk_nginx {
	# script "killall -0 nginx"  script的第一种写法：脚本执行的返回结果，改变优先级，keepalived继续发送通告消息，backup比较优先级再决定
	script "/etc/keepalived/check_nginx.sh"  #script第二种写法：脚本里面检测到异常，直接关闭keepalived进程，backup机器接收不到advertisement会抢占IP。异常时exit 1，正常退出exit 0。如果脚本执行结果为0，并且weight配置的值大于0，则优先级相应的增加；如果脚本执行结果非0，并且weight配置的值小于0，则优先级相应的减少
	interval 2 # 每2s检测一次 
	weight -5  # 检测失败（脚本返回非0）则优先级 -5
	fall 3 # 检测连续 2 次失败才算确定是真失败。会用weight减少优先级（1-255之间）
	rise 2 #检测 1 次成功就算成功。但不修改优先级
}
vrrp_instance VI_1 {
	state MASTER  # 指定instance的初始状态，最后会通过竞选通过优先级来确定（这里设置为MASTER，在发送通告时，同时发送自己的优先级，另外一台发现优先级不如自己的高，那么另一台会就回抢占为MASTER）
	interface eth0 #实例绑定的网卡，因为在配置虚拟IP的时候必须是在已有的网卡上添加
	mcast_src_ip 172.29.88.224  #发送多播数据包时的源IP地址，心跳端口。若没有设置，默认个为interface指定IP
	virtual_router_id 51 #设置VRID，这里非常重要，相同的VRID为一个组，决定多播的MAC地址
	priority 101 #设置本节点的优先级，优先级高的为master
	advert_int 2 #检查间隔，默认为1秒。这就是VRRP的定时器，MASTER每隔这样一个时间间隔，就会发送一个advertisement报文以通知组内其他路由器自己工作正常
	authentication { #定义认证方式和密码，主从必须一样
		auth_type PASS
		auth_pass 1111
	}
	virtual_ipaddress {#这里设置的就是VIP，也就是虚拟IP地址，他随着state的变化而增加删除，当state为master的时候就添加，当state为backup的时候删除，这里主要是有优先级来决定的，和state设置的值没有多大关系，这里可以设置多个IP地址
		172.29.88.222
	}
	track_script { #引用VRRP脚本，即在 vrrp_script 部分指定的名字。定期运行它们来改变优先级，并最终引发主备切换。
		chk_nginx 
	}
}
```

```
#全局定义块
global_defs {
    # 邮件通知配置
    notification_email {
        email1
        email2
    }
    notification_email_from email
    smtp_server host
    smtp_connect_timeout num

    lvs_id string
    router_id string    ## 标识本节点的字条串,通常为hostname
}

#VRRP 实例定义块
vrrp_sync_group string { 
    group {
        string
        string
    }
}

vrrp_instance string {
    state MASTER|BACKUP
    virtual_router_id num
    interface string
    mcast_src_ip @IP 
    priority num
    advert_int num
    nopreempt
    smtp_alert
    lvs_sync_daemon_interface string 
    authentication {
        auth_type PASS|AH
        auth_pass string
    }

    virtual_ipaddress {  # Block limited to 20 IP addresses @IP
        @IP
        @IP
    }
}

#虚拟服务器定义块
virtual_server (@IP PORT)|(fwmark num) { 
    delay_loop num
    lb_algo rr|wrr|lc|wlc|sh|dh|lblc 
    lb_kind NAT|DR|TUN
    persistence_timeout num 
    protocol TCP|UDP
    real_server @IP PORT { 
        weight num
        notify_down /path/script.sh
        TCP_CHECK { 
            connect_port num 
            connect_timeout num
        }
    }

    real_server @IP PORT {
        weight num
        MISC_CHECK {
            misc_path /path_to_script/script.sh(or misc_path “/path_to_script/script.sh <arg_list>”)
        }
    }

    real_server @IP PORT {
        weight num
        HTTP_GET|SSL_GET {
            url { 
                # You can add multiple url block path alphanum
                digest alphanum
            }
            connect_port num
            connect_timeout num 
            nb_get_retry num 
            delay_before_retry num
        }
    }
}

全局定义块
1、email通知（notification_email、smtp_server、smtp_connect_timeout）：用于服务有故障时发送邮件报警，可选项，不建议用。需要系统开启sendmail服务，建议用第三独立监控服务，如用nagios全面监控代替。
2、lvs_id：lvs负载均衡器标识，在一个网络内，它的值应该是唯一的。
3、router_id：用户标识本节点的名称，通常为hostname
4、花括号｛｝：用来分隔定义块，必须成对出现。如果写漏了，keepalived运行时不会得到预期的结果。由于定义块存在嵌套关系，因此很容易遗漏结尾处的花括号，这点需要特别注意。

VRRP实例定义块
vrrp_sync_group：同步vrrp级，用于确定失败切换（FailOver）包含的路由实例个数。即在有2个负载均衡器的场景，一旦某个负载均衡器失效，需要自动切换到另外一个负载均衡器的实例是哪
group：至少要包含一个vrrp实例，vrrp实例名称必须和vrrp_instance定义的一致

vrrp_instance：vrrp实例名
1> state：实例状态，只有MASTER 和 BACKUP两种状态，并且需要全部大写。抢占模式下，其中MASTER为工作状态，BACKUP为备用状态。当MASTER所在的服务器失效时，BACKUP所在的服务会自动把它的状态由BACKUP切换到MASTER状态。当失效的MASTER所在的服务恢复时，BACKUP从MASTER恢复到BACKUP状态。
2> interface：对外提供服务的网卡接口，即VIP绑定的网卡接口。如：eth0，eth1。当前主流的服务器都有2个或2个以上的接口（分别对应外网和内网），在选择网卡接口时，一定要核实清楚。
3> mcast_src_ip：本机IP地址
4> virtual_router_id：虚拟路由的ID号，每个节点设置必须一样，可选择IP最后一段使用，相同的 VRID 为一个组，他将决定多播的 MAC 地址。
5> priority：节点优先级，取值范围0～254，MASTER要比BACKUP高
6> advert_int：MASTER与BACKUP节点间同步检查的时间间隔，单位为秒
7> lvs_sync_daemon_inteface：负载均衡器之间的监控接口,类似于 HA HeartBeat 的心跳线。但它的机制优于 Heartbeat，因为它没有“裂脑”这个问题,它是以优先级这个机制来规避这个麻烦的。在 DR 模式中，lvs_sync_daemon_inteface与服务接口interface使用同一个网络接口
8> authentication：验证类型和验证密码。类型主要有 PASS、AH 两种，通常使用PASS类型，据说AH使用时有问题。验证密码为明文，同一vrrp 实例MASTER与BACKUP使用相同的密码才能正常通信。
9> smtp_alert：有故障时是否激活邮件通知
10> nopreempt：禁止抢占服务。默认情况，当MASTER服务挂掉之后，BACKUP自动升级为MASTER并接替它的任务，当MASTER服务恢复后，升级为MASTER的BACKUP服务又自动降为BACKUP，把工作权交给原MASTER。当配置了nopreempt，MASTER从挂掉到恢复，不再将服务抢占过来。
11> virtual_ipaddress：虚拟IP地址池，可以有多个IP，每个IP占一行，不需要指定子网掩码。注意：这个IP必须与我们的设定的vip保持一致。

虚拟服务器virtual_server定义块
virtual_server：定义一个虚拟服务器，这个ip是virtual_ipaddress中定义的其中一个，后面一个空格，然后加上虚拟服务的端口号。
1> delay_loop：健康检查时间间隔，单位：秒
2> lb_algo：负载均衡调度算法，互联网应用常用方式为wlc或rr
3> lb_kind：负载均衡转发规则。包括DR、NAT、TUN 3种，一般使用路由（DR）转发规则。
4> persistence_timeout：http服务会话保持时间，单位：秒
5> protocol：转发协议，分为TCP和UDP两种
real_server：真实服务器IP和端口，可以定义多个
1> weight：负载权重，值越大，转发的优先级越高
2> notify_down：服务停止后执行的脚本
3> TCP_CHECK：服务有效性检测
* connect_port：服务连接端口
* connect_timeout：服务连接超时时长，单位：秒
* nb_get_retry：服务连接失败重试次数
* delay_before_retry：重试连接间隔，单位：秒

```
#### nginx配置
2台nginx服务器上的配置应该是完全一样的，nginx.conf 里面的 server_name 尽量使用域名来代替，然后dns解析这个域名到虚拟IP 172.29.88.222。

docker启动nginx,进行配置，要把配置文件的目录挂载出来,但是nginx却是先加载一个主配置文件nginx.conf，在nginx.conf里再加载conf.d目录下的子配置文件（一般最少一个default.conf文件）。

[docker下安装nginx](https://segmentfault.com/a/1190000015758373)
```

```


#### 监控nginx的脚本
nginx监控脚本1
检测ngnix的运行状态，并在nginx进程不存在时尝试重新启动ngnix，如果启动失败则停止keepalived，准备让其它机器接管
```shell /etc/keepalived/check_nginx.sh
#!/bin/bash
counter=$(ps -C nginx --no-heading|wc -l)
if [ "${counter}" = "0" ]; then
/usr/local/bin/nginx
sleep 2
counter=$(ps -C nginx --no-heading|wc -l)
if [ "${counter}" = "0" ]; then
/etc/init.d/keepalived stop
fi
fi
```
nginx监控脚本2 
curl 主页连续2个5s没有响应则切换
```
#!/bin/bash
# curl -IL http://localhost/member/login.htm
# curl --data "memberName=fengkan&password=22" http://localhost/member/login.htm
count = 0
for (( k=0; k<2; k++ )) 
do 
check_code=$( curl --connect-timeout 3 -sL -w "%{http_code}\\n" http://localhost/login.html -o /dev/null )
if [ "$check_code" != "200" ]; then
count = count +1
continue
else
	count = 0
break
fi
done
if [ "$count" != "0" ]; then
# /etc/init.d/keepalived stop
exit 1
else
exit 0
fi
```




#### 安装配置遇到的问题
运行'./configure --prefix=/usr/local/keepalived'报错

>configure: error:
  !!! OpenSSL is not properly installed on your system. !!!
  !!! Can not include OpenSSL headers files.            !!!cd

解决方法，执行以下命令即可：
>sudo apt-get install libssl-dev

再执行'./configure --prefix=/usr/local/keepalived'

执行 设置keepalived服务开机启动'chkconfig keepalived on'报错
>bash: chkconfig: command not found

解决方法是使用替换的sysv-rc-conf 
>$ apt-get install sysv-rc-conf
>$ sysv-rc-conf keepalived on

keepalived正常启动但是虚IP（VIP）没有生成的问题
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_script check_haproxy {
   script "/etc/keepalived/check_nginx.sh"
   interval 3
}

vrrp_instance VI_1 {
    state MASTER
    interface enp129s0f0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.3.2/24
    }
}

```
>1.脚本如上正常编写后，启动keepalived正常，但是查看虚拟ip: ip addr | grep "192.168.3.2/24"，无结果
>2.检查脚本括号是否完整（完成）
>3.查看keepalived的日志文件路径/var/log/syslog，“ keepalived.conf 配置文件中将 virtual_router_id 参数设置了默认id——51，然后同一内网内还有其他keepalived集群也设置了51这个默认路由id，因此keepalived爆“目前xxx这个虚拟IP不能绑定到51这个路由id上”的错误”（https://www.zifangsky.cn/994.html），
>4.修改    virtual_router_id为75

### 参考链接
[keepalived + nginx 初步实现高可用](https://klionsec.github.io/2017/12/23/keepalived-nginx/)

[Nginx+Keepalived实现站点高可用](https://cloud.tencent.com/developer/article/1124764)

[Nginx+Keepalived(双机热备)搭建高可用负载均衡环境(HA) ](https://my.oschina.net/xshuai/blog/917097)

### 测试用例
Nginx为什么会停止响应呢？有以下几种情况：

1、Nginx的所有进程被强行终止（或管理进程）。这种情况，是我们需要检查和切换的。无论什么情况下进程被终止了，如果它不能重启，我们就要切换到备机。

> Blockquote

2、Nginx日志盘的挂载点崩溃或者磁盘写满。这个也是我们需要检查和切换的。

3、Nginx已经达到设置的最大连接数，暂时停止响应。这种情况下，我们不能进行备机切换，因为通过VIP:172.16.200.3连接过来的用户请求比较多（在我们优化参数后，可以达到65535 / 4的数量），一旦我们进行备机切换，这些用户请求将全部异常。这个问题的解决需要靠增加负载机器，而不是主备切换。

4、Nginx物理机异常关机。这个肯定是需要进行检查和切换的。

## 后期编排
### docker + nginx + keepalived 的镜像包
[1.githubdemo](https://github.com/sonicrang/Docker-Keepalived-Nginx)

[2.实践Keepalived+Nginx实现高可用Web服务](http://www.longmon.net/a/ha-web-with-keepalived.html)

[3.Docker+keepalived+nginx实现主从热备](https://juejin.im/post/5dc517386fb9a04a9272110b)

[4.Docker容器与宿主机同网段互相通信](http://www.louisvv.com/archives/695.html)



### 终极版本
#### 下载centos基础镜像
docker pull centos:7.6.1810

#### 编辑Dockerfile
Dockerfile中将前端的配置文件放入容器nginx配置中

```
FROM centos:7.6.1810
RUN yum install keepalived -y
COPY keepalived.conf /etc/keepalived/
COPY keepalived_init.sh /etc/keepalived/
COPY check_nginx.sh /etc/keepalived/
RUN chmod a+x /etc/keepalived/check_nginx.sh
RUN yum install yum-utils -y
COPY nginx.repo etc/yum.repos.d/
RUN yum-config-manager --enable nginx-mainline -y
RUN yum install nginx -y
ENV WORK_PATH /etc/nginx
ENV CONF_FILE_NAME nginx.conf
RUN rm $WORK_PATH/$CONF_FILE_NAME
COPY ./$CONF_FILE_NAME $WORK_PATH/
COPY ./dev/ $WORK_PATH/dev/
#COPY ./build/. $WORK_PATH/build/
#COPY ./ybin-build/. $WORK_PATH/ybin-build/
#COPY ./gis-build/. $WORK_PATH/gis-build/
RUN chmod a+r $WORK_PATH/$CONF_FILE_NAME
RUN chmod a+x /etc/keepalived/keepalived_init.sh
CMD ["/etc/keepalived/keepalived_init.sh"]                            
```

#### 编辑keepalived.conf
```
! Configuration File for keepalived
global_defs {
    router_id KEEPALIVED_ROUTE_ID
    script_user root
    enable_script_security
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight 2
}

vrrp_instance VI_1 {
    state KEEPALIVED_STATE
    interface KEEPALIVED_INTERFACE
    virtual_router_id 151
    priority KEEPALIVED_PRIORITY
    advert_int 2
    #unicast_peer {
    #    192.168.237.139
    #}
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        KEEPALIVED_VIP
    }
    track_script {
       check_nginx
    }
}
```
#### 编辑check_nginx.sh
```
#!/bin/bash
nginx_status=$(ps aux | grep -Ev "grep|$0" | grep '\bnginx\b' | wc -l)
if [ $nginx_status -lt 1 ];then
    ps -ef | grep keepalived | grep -v grep | grep -v '/bin/bash' | awk -F ' +' '{print $2}' | xargs -I '{}' kill -9 {};
fi
```
#### 编辑nginx.repo
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
#### 编辑keepalived_init.sh
```
#!/bin/bash
sed -i "s/KEEPALIVED_ROUTE_ID/$KEEPALIVED_ROUTE_ID/g" /etc/keepalived/keepalived.conf;
sed -i "s/KEEPALIVED_STATE/$KEEPALIVED_STATE/g" /etc/keepalived/keepalived.conf;
sed -i "s/KEEPALIVED_INTERFACE/$KEEPALIVED_INTERFACE/g" /etc/keepalived/keepalived.conf;
sed -i "s/KEEPALIVED_PRIORITY/$KEEPALIVED_PRIORITY/g" /etc/keepalived/keepalived.conf;
sed -i "s/KEEPALIVED_VIP/$KEEPALIVED_VIP/g" /etc/keepalived/keepalived.conf;
sed -i "s/SERVERS/$SERVERS/g" /etc/nginx/dev/web-park.conf;
nginx;
keepalived -n -d -D -f /etc/keepalived/keepalived.conf --log-console
```
#### 构建镜像
docker build -t nginx_keepalived_ha .

#### 运行镜像
需要在镜像设置环境变量，使用初始化脚本初始化，替换keepalived.conf配置文件中的字段
- 主要KEEPALIVED_INTERFACE 对应与本机ip去映射虚拟ip
- ip|ip1|ip2 对应为nginx的配置文件中使用的网关ip
- vip 为自行设置的虚拟ip，需要和主机ip在同一个网段
```
主节点：
docker run --privileged --network host -e host=ip -e SERVERS="server ip1; server ip2;" -e KEEPALIVED_ROUTE_ID=master -e KEEPALIVED_STATE=MASTER -e KEEPALIVED_INTERFACE=en0 -e KEEPALIVED_PRIORITY=100 -e KEEPALIVED_VIP=vip nginx_keepalived_ha


备节点：
docker run --privileged --network host -e host=ip -e SERVERS="server ip1; server ip2;" -e KEEPALIVED_ROUTE_ID=backup -e KEEPALIVED_STATE=BACKUP -e KEEPALIVED_INTERFACE=en0 -e KEEPALIVED_PRIORITY=99 -e KEEPALIVED_VIP=vip nginx_keepalived_ha

```
#### 注意事项：
1.需要在宿主机上执行ipvsadm(LVS虚拟服务器的管理工具)命令，如果这两个命令不存在，按照下面的命令先安装：
```
centos: yum install ipvsadm -y

ubuntu: apt-get install ipvsadm -y
```
2.在做服务部署前需要分配一个VIP
3.在部署前需要提前知道宿主机的网卡名称
4.使用devops则需要些编排文件
5.devops部署
```
sudo docker save -o nginx-keepalived-ha-v1.tar ampregistry:5000/nginx-keepalived-ha-v1

sudo devops-svcctl load -p nginx-keepalived-ha-v1.tar 
```

