---
title: 网关层接入Https
tags: ssl,springcloud
renderNumberedHeading: true
grammar_cjkRuby: true
---

格言：好事多磨

## Why
- http协议访问接口，最大的问题就是用fiddler这样的抓包工具，直接就能看到明文的访问url，参数列表和参数值。
- 随意抓取我们的数据，然后很轻松地制造山寨产品。
- 随意修改请求中的参数值来尝试攻击和破解我们的后端接口，找到侵入点，进行攻击。
## How
1. 将http请求改造为https请求
2. 将要访问的api接口作为请求参数传递给app后端系统，后端系统重新拼接url后访问接口，并且原请求中的参数，除了api接口之外，必须原样带过去。
3. 返回的数据经过加密处理后给到app。
## Action
### 将http请求改造为https请求
>（1）要有一个SSL证书，证书怎么获取呢？买（通过证书授权机构购买）或者自己生成（通过keytool生成）。
（2）在spring boot中启用HTTPS
（3）将HTTP重定向到HTTPS（可选）

#### ssl证书
1.[ssl证书格式了解](https://blog.freessl.cn/ssl-cert-format-introduce/)：PEM、CER、JKS、PKCS12
>Java Key Storage，很容易知道这是 JAVA 的专属格式，利用 JAVA 的一个叫 keytool 的工具可以进行格式转换。一般用于 Tomcat 服务器。

2.如何生成证书
（1）自己通过keytool生成；
```
keytool -genkey -alias tomcat -dname "CN=Andy,OU=kfit,O=kfit,L=HaiDian,ST=BeiJing,C=CN"  -storetype PKCS12 -keyalg RSA -keysize 2048  -keystore keystore.p12 -validity 365 

-genkey：生成key；
-alias：key的别名；
-dname：指定证书拥有者信息
-storetype：密钥库的类型为JCEKS。常用的有JKS(默认),JCEKS(推荐),PKCS12,BKS,UBER。每个密钥库只可以是其中一种类型。
-keyalg：DSA或RSA算法(当使用-genkeypair参数)，DES或DESede或AES算法(当使用-genseckey参数)；
-keysize：密钥的长度为512至1024之间(64的倍数)
-keystore：证书库的名称
-validity：指定创建的证书有效期多少天

dname的值详解：
　　CN(Common Name名字与姓氏)
　　OU(Organization Unit组织单位名称)
　　O(Organization组织名称)
　　L(Locality城市或区域名称)
　　ST(State州或省份名称)
```

```

#1.
keytool -genkey -alias tomcat -dname "CN=Andy,OU=kfit,O=kfit,L=HaiDian,ST=BeiJing,C=CN"  -storetype PKCS12 -keyalg RSA -keypass 123456 -keysize 2048  -validity 36500 -keystore test.keystore -storepass 123456
#2.
keytool -importkeystore -srcstoretype JKS -srckeystore test.keystore -srcstorepass 123456 -srcalias tomcat -srckeypass 123456 -deststoretype PKCS12 -destkeystore test.p12 -deststorepass 123456 -destalias client -destkeypass 123456 -noprompt
#3.
openssl pkcs12 -in test.p12 -nocerts -nodes -out test.key
#4.
keytool -export -alias tomcat -keystore test.keystore -rfc -file test.cer
#5.
keytool -importkeystore -srcstoretype JKS -srckeystore test.keystore -srcstorepass 123456 -srcalias tomcat -srckeypass 123456 -deststoretype PKCS12 -destkeystore test.p12 -deststorepass 123456 -destalias test -noprompt


openssl req -x509 -in htx-server.csr -CA zydc.crt CAkey zydc.pem -CAcreateserial -days 3650 -out htx-server.crt
```
（2）通过证书授权机构购买
3.证书存放路径：src->main->resource下，注意springboot和springcloud的版本...
#### 网关层改造
1.gateway->application.properties
```application.properties
#https端口号.  
server.port: 1101  
#证书的路径.  
server.ssl.key-store: classpath:keystore.p12  
#证书密码，请修改为您自己证书的密码.  
server.ssl.key-store-password: 123456  
#秘钥库类型  
server.ssl.keyStoreType: PKCS12  
#证书别名  
server.ssl.keyAlias: tomcat 
```
2.[postman调用https设置证书](https://learning.postman.com/docs/postman/sending-api-requests/certificates/)，注意postman版本

#### 同时支持http和https

#### 将http请求重定向到https

## 参考资料：
[1.springcloud zuul配置https实现安全的app接口访问](https://blog.csdn.net/johntsu2006/article/details/80723726)

[2.Spring Boot 使用SSL-HTTPS【从零开始学Spring Cloud】](https://www.iteye.com/blog/412887952-qq-com-2400846)

[3.SSL 证书格式普及，PEM、CER、JKS、PKCS12](https://blog.freessl.cn/ssl-cert-format-introduce/)

[4.证书格式转换](https://myssl.com/cert_convert.html)

[5.api接口安全以及https普及](https://www.cnblogs.com/applelife/p/10488177.html)

[6.spring-cloud-gateway使用https注意事项2---如何在转发后端服务的时候使用http](https://www.jianshu.com/p/5a36129399f2)

[7.Java 和 HTTP 的那些事（四） HTTPS 和 证书](https://www.aneasystone.com/archives/2016/04/java-and-https.html)

[8.springboot2.0配置Https访问](https://www.itbo.top/springboot/2019/03/25/springboot-config-https.html)

[9.科普：TLS、SSL、HTTPS以及证书](https://www.cnblogs.com/kyrios/p/tls-and-certificates.html)

[10.SSL证书在线工具SSL Online Tools](https://www.chinassl.net/ssltools/)

[11.CSR在线生成工具](https://myssl.com/csr_create.html)

[12.keystore导出p12,cer,crt,.key.pem证书文件格式](https://blog.csdn.net/lijun169/article/details/89141073)

[13.OpenSSL命令---pkcs12](https://blog.csdn.net/as3luyuan123/article/details/16105475)

[14.openssl 生成keystore](https://blog.csdn.net/luyunquan1988/article/details/53763370)

[15.postman调用https设置证书](https://learning.postman.com/docs/postman/sending-api-requests/certificates/)