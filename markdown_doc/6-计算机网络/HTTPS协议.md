---
title: HTTPS协议
tags: 计算机网络, 应用层
renderNumberedHeading: true
grammar_cjkRuby: true
---
参考资料
- [x] [HTTPS协议](https://blog.csdn.net/qq_36711489/article/details/92074350)
- [ ] [HTTP和HTTPS协议，看一篇就够了](https://blog.csdn.net/xiaoming100001/article/details/81109617)
- [ ] [简书-https协议](https://www.jianshu.com/p/f9b8a3e62af1)
- [ ] [wiki-HTTPS原理剖析](https://wiki.megvii-inc.com/pages/viewpage.action?pageId=126223027)
- [ ] [nginx反向代理WebSocket](https://www.xncoding.com/2018/03/12/fullstack/nginx-websocket.html)

[toc]
## 概念
HTTPS是在HTTP的基础上添加了SSL层，通过**传输加密和身份认证**保证传输过程中数据的安全性。
## 原理
HTTPS  = HTTP + SSL/TLS
- HTTP，客户端和服务器通过TCP建立连接后，客户端通过HTTP协议发送请求和接收服务端返回的请求。
- SSL(Secure Sockets Layer 安全套接层),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层TCP与应用层HTTP之间对网络连接进行加密。[百度百科]
![enter description here](./images/1577112311794.png)
## 安全机制
- 证书：通过第三方权威证书颁发机构（如VeriSign）验证和担保网站的身份，防止他人伪造网站身份与不知情的用户建立加密连接。
- 密钥交换：通过公钥（非对称）加密在网站服务器和用户之间协商生成一个共同的会话密钥。
- 会话加密：通过机制(2)协商的会话密钥，用对称加密算法对会话的内容进行加密。
- 消息校验：通过消息校验算法来防止加密信息在传输过程中被篡改。

## 缺点
- 安全性范围有限度，不能保证绝对安全
- SSL加密和解密增加了额计算资源消耗
- 同样网络环境下，通信变慢
## 配置
## 方案
 [wiki-devops平台https证书管理](https://wiki.megvii-inc.com/pages/viewpage.action?pageId=34887052)
 
1.使用证书的两种方式：
- 用户手动上传HTTPS的*.crt和*.key文件，可能需要自行生成。
- devops资料显示;
>若集群拓扑结构发生变化，可以手工生成DevOps 默认证书，在DevOps Manager所在节点，输入：sudo devops-manager -replaceBuiltinHttpsCert -genHttpsCert   , 然后到集群页面点击配置同步，重启应用后就会使用新的证书。

2.devops设置开启http
![挂载http证书](./images/1577173123760.png)
3.重新部署应用
4.修改nginx的配置
![nginx配置](./images/1577173043898.png)
5.yml文件修改 enable_https_cert:true

## 步骤

1. 打开前端devops的挂载http，后面通过编排文件添加参数 （enable_https_cert: true）

![devops修改实例](./images/1577351781950.png)

2.修改前端nginx.config的配置文件
```
# 1.文件在容器对应路径下
ssl_certificate /etc/ssl/devops/https_cert_crt;
ssl_certificate_key /etc/ssl/devops/https_cert_key;
# 2. 监听的端口后面添加ssl
```
3.重新jekins部署前端代码
4.解决不安全链接 
- 描述
```
Failed to construct 'WebSocket': An insecure WebSocket connection may not be initiated from a page loaded over HTTPS.
```
- 方案：
[1.无法通过HTTPS加载的页面启动不安全的WebSocket连接](https://blog.csdn.net/weixin_30693683/article/details/95139180)
[2.NGINX反向代理websocket并启用SSL](https://stackoverflow.com/questions/12102110/nginx-to-reverse-proxy-websockets-and-enable-ssl-wss)
- 步骤：
1.webSocket几个链接
2.nginx 添加location
- 配置参考
```
location / {
    # redirect all HTTP traffic to localhost:8080
    proxy_pass http://localhost:8080;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # WebSocket support
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```
- nginx的配置正则表达式
- ~* 为不区分大小写匹配
```
# action 1 匹配/passTodayComWebsocket/
location /passTodayComWebsocket/ {
        proxy_pass http://$host:2225;
        proxy_pass_header Referer;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        # WebSocket support
		proxy_read_timeout     60;
        proxy_connect_timeout  60;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
prob1.进首页还是ws
prob2.ERR_CONNECTION_TIMED_OUT

# action 2.配置修改
 location ~*/*Websocket/ {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://$host:2225;
        proxy_http_version 1.1;
        proxy_ssl_certificate     /etc/ssl/devops/https_cert_crt;
        proxy_ssl_certificate_key /etc/ssl/devops/https_cert_key;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }


    location ~/websocket/ {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://$host:2228;
        proxy_http_version 1.1;
        proxy_ssl_certificate     /etc/ssl/devops/https_cert_crt;
        proxy_ssl_certificate_key /etc/ssl/devops/https_cert_key;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```
![nginx配置](./images/1577521603744.png)

![websocket链接不上](./images/1577521556024.png)
方案：[前端配合修改](https://stackoverflow.com/questions/41381444/websocket-connection-failed-error-during-websocket-handshake-unexpected-respon)
3.前端ws修改为wss:[How to Proxy WSS WebSockets with NGINX](https://www.serverlab.ca/tutorials/linux/web-servers-linux/how-to-proxy-wss-websockets-with-nginx/)

- websocket问题修复action
1.创建新的conf文件
```websocket.conf
server {
    listen 3335 ssl;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_ssl_certificate     /etc/ssl/devops/https_cert_crt;
    proxy_ssl_certificate_key /etc/ssl/devops/https_cert_key;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    location / {
        proxy_pass http://$host:2225;
    }

    location ^~ /structure/overview/ {
        proxy_pass http://$host:2228;
    }
}

```
2.前端创建websocket 连接
- ws修改成wss
- 端口号映射成3335

3.最后修复问题ERR_CONNECTION_TIMED_OUT，定位是端口号没有进行映射，之前放在8200下的，通过'wss://ip:8888/recognitionListWebsocket/{uuid}'应该能实现访问的。

4.工具
- [正则表达式测试工具](https://c.runoob.com/front-end/854)
- [websocket在线测试工具](http://coolaf.com/tool/chattest)

到此安全漏洞修复完成，特别感谢liutianwei和guanxipeng两位小哥哥耐心指导[笔芯~]