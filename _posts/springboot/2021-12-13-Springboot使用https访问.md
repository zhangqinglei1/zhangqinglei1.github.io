---
layout: post
title:  "Springboot使用https访问"
date:   2021-12-13 21:14:54
categories: SpringBoot
tags: SpringBoot
excerpt: Springboot使用https访问
mathjax: true
---

* content
{:toc}
## HTTPS介绍

HTTPS 是http的安全访问协议。

HTTPS 协议是由 HTTP 加上 [TLS](https://baike.baidu.com/item/TLS/2979545)/[SSL](https://baike.baidu.com/item/SSL/320778) 协议构建的可进行加密传输、身份认证的网络协议，主要通过[数字证书](https://baike.baidu.com/item/数字证书/326874)、[加密算法](https://baike.baidu.com/item/加密算法/2816213)、非对称密钥等技术完成互联网数据传输加密，实现互联网传输安全保护。设计目标主要有三个。

（1）数据保密性：保证数据内容在传输的过程中不会被第三方查看。就像快递员传递包裹一样，都进行了封装，别人无法获知里面装了什么 [4] 。

（2）数据完整性：及时发现被第三方篡改的传输内容。就像快递员虽然不知道包裹里装了什么东西，但他有可能中途掉包，数据完整性就是指如果被掉包，我们能轻松发现并拒收 [4] 。

（3）身份校验安全性：保证数据到达用户期望的目的地。就像我们邮寄包裹时，虽然是一个封装好的未掉包的包裹，但必须确定这个包裹不会送错地方，通过身份校验来确保送对了地方 [4] 。

SSL证书目前市面上有以下几种

- .DER .CER，文件是二进制格式，只保存证书，不保存私钥。
- .PEM，一般是文本格式，可保存证书，可保存私钥。
- .CRT，可以是二进制格式，可以是文本格式，与 .DER 格式相同，不保存私钥。
- .PFX .P12，二进制格式，同时包含证书和私钥，一般有密码保护。
- .JKS，二进制格式，同时包含证书和私钥，一般有密码保护。

JKS是利用JDK来生产的一组秘钥

## SpringBoot配置SSL协议

SpringBoot可以通过在application.properties或application.yml配置文件中配置各种server.ssl.*属性来声明性使用SSL（https），

比如下面的例子使用了JKS证书

```
server:
  ssl:
    # 证书路径
    key-store: classpath:server.keystore
    key-alias: tomcat
    enabled: true
    //证书的类型
    key-store-type: JKS
    #与申请时输入一致
    key-store-password: 123456
  # 浏览器默认端口 和 80 类似
  port: 443
```

比如下面的例子使用了pkcs12协议

```
server:
  ssl:
    # 证书路径
    key-store: classpath:server.keystore
    key-alias: tomcat
    enabled: true
    //证书的类型
    key-store-type: pkcs12
    #与申请时输入一致
    key-store-password: 123456
  # 浏览器默认端口 和 80 类似
  port: 443
```

SpringBoot无法同时使得Http和Https协议共存，如果要同时支持，可以通过如下介绍的编程来进行同时支持，具体在下面会讲。

使用JDK工具生成证书

在jdk的bin目录下，通过keytool进行生生成

## 证书相关

### Keytools生成证书

 JAVA 的专属格式，一般用于 Tomcat 服务器。

在jdk/bin目录下执行如下

```
C:\Program Files\Java\jdk1.8.0_251\bin>keytool
密钥和证书管理工具
命令:
 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令
使用 "keytool -command_name -help" 获取 command_name 的用法
```

主要使用genkeypair来生产证书

```
C:\Program Files\Java\jdk1.8.0_251\bin>keytool -genkeypair -help
keytool -genkeypair [OPTION]...

生成密钥对

选项:

 -alias <alias>                  要处理的条目的别名
 -keyalg <keyalg>                密钥算法名称
 -keysize <keysize>              密钥位大小
 -sigalg <sigalg>                签名算法名称
 -destalias <destalias>          目标别名
 -dname <dname>                  唯一判别名
 -startdate <startdate>          证书有效期开始日期/时间
 -ext <value>                    X.509 扩展
 -validity <valDays>             有效天数
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```

因此生成证书可以使用

```
keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keystore "server.keystore" 
```

参数说明：

```
-genkeypair：生成一对非对称密钥;

-alias：指定密钥对的别名，该别名是公开的;
-keyalg：指定加密算法，本例中的采用通用的RAS加密算法;
-keystore:密钥库的路径及名称，不指定的话，默认在操作系统的用户目录下生成一个".keystore"的文件
```

备注1：关于JDK支持的加密算法，请查看[https://docs.oracle.com/javase/8/docs/technotes/guides/security/SunProviders.html](https://docs.oracle.com/javase/8/docs/technotes/guides/security/SunProviders.html){:target="_blank"}

备注2：如果你的JDK在C盘，CMD需要使用管理员权限。

注意：

　　**1.密钥库的密码至少必须6个字符，可以是纯数字或者字母或者数字和字母的组合等等**

　　**2."名字与姓氏"一般指的是域名，而不是我们的个人姓名，其他的可以不填**

　　**3.最后输入y或者yes均可**

执行完上述命令后，在bin目录下生成了一个"server.keystore"的文件

使用SpringBoot进行配置即可访问Https。

生成完成后，会有这样的一行提示信息

```
Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore server.keystore -destkeystore server.keystore -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
```

这说明PKCS12才是行业标准格式，我们需要迁移到行业标准格式

```
keytool -importkeystore -srckeystore server.keystore -destkeystore server.keystore -deststoretype pkcs12
```

生成后会对旧的keystore文件进行备份，生成old文件，分别进行查看如下



```
keytool -list -keystore XXX.keystore
C:\Program Files\Java\jdk1.8.0_251\bin>keytool -list -keystore server.keystore
输入密钥库口令:
密钥库类型: PKCS12
密钥库提供方: SUN

您的密钥库包含 1 个条目

tomcat, 2021-12-13, PrivateKeyEntry,
证书指纹 (SHA1): AF:26:10:AA:60:30:DE:58:E9:D0:05:93:74:69:C4:7D:F8:C7:84:3C

C:\Program Files\Java\jdk1.8.0_251\bin>keytool -list -keystore server.keystore.old
输入密钥库口令:
密钥库类型: jks
密钥库提供方: SUN

您的密钥库包含 1 个条目

tomcat, 2021-12-13, PrivateKeyEntry,
证书指纹 (SHA1): AF:26:10:AA:60:30:DE:58:E9:D0:05:93:74:69:C4:7D:F8:C7:84:3C

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore server.keystore.old -destkeystore server.keystore.old -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
```

### 导出证书

比如将server.keystore导出到server.crt中

```
keytool -export -alias tomcat -file server.crt -keystore server.keystore
```

这样就生成了文件server.crt

### 导出证书

比如将server.crt导出到test_cacerts证书库

```
keytool -import -keystore test_cacerts -file server.crt

C:\Program Files\Java\jdk1.8.0_251\bin>keytool -import -keystore test_cacerts -file server.crt
输入密钥库口令:
再次输入新口令:
所有者: CN=www.test.com, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
发布者: CN=www.test.com, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
序列号: 1186e728
有效期为 Mon Dec 13 22:50:37 CST 2021 至 Sun Mar 13 22:50:37 CST 2022
证书指纹:
         MD5:  37:A1:6E:BA:C2:A2:0C:A6:D8:B0:42:8A:E6:55:14:32
         SHA1: AF:26:10:AA:60:30:DE:58:E9:D0:05:93:74:69:C4:7D:F8:C7:84:3C
         SHA256: 14:71:D4:C4:17:EB:10:A3:61:1E:2E:9D:3A:12:8A:DA:F1:81:E3:90:AE:A8:83:82:6E:E9:E9:B8:03:DB:E6:5B
签名算法名称: SHA256withRSA
主体公共密钥算法: 2048 位 RSA 密钥
版本: 3

扩展:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 17 1D BE 58 5A 7E E6 2D   39 4F 4B 16 5F 87 03 AC  ...XZ..-9OK._...
0010: 81 15 3E 94                                        ..>.
]
]

是否信任此证书? [否]:  y
证书已添加到密钥库中
```

### 查看证书信息

```
keytool -printcert -file "server.crt"
keytool -printcert -file "****"
C:\Program Files\Java\jdk1.8.0_251\bin>keytool -printcert -file "server.crt"
所有者: CN=www.test.com, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
发布者: CN=www.test.com, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
序列号: 1186e728
有效期为 Mon Dec 13 22:50:37 CST 2021 至 Sun Mar 13 22:50:37 CST 2022
证书指纹:
         MD5:  37:A1:6E:BA:C2:A2:0C:A6:D8:B0:42:8A:E6:55:14:32
         SHA1: AF:26:10:AA:60:30:DE:58:E9:D0:05:93:74:69:C4:7D:F8:C7:84:3C
         SHA256: 14:71:D4:C4:17:EB:10:A3:61:1E:2E:9D:3A:12:8A:DA:F1:81:E3:90:AE:A8:83:82:6E:E9:E9:B8:03:DB:E6:5B
签名算法名称: SHA256withRSA
主体公共密钥算法: 2048 位 RSA 密钥
版本: 3

扩展:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 17 1D BE 58 5A 7E E6 2D   39 4F 4B 16 5F 87 03 AC  ...XZ..-9OK._...
0010: 81 15 3E 94                                        ..>.
]
]

```

### 删除密钥库中的条目

删除server.keystore中别名为tomcat的证书条目

```
keytool -delete -keystore server.keystore -alias tomcat
C:\Program Files\Java\jdk1.8.0_251\bin>keytool -delete -keystore server.keystore -alias tomcat
输入密钥库口令:
C:\Program Files\Java\jdk1.8.0_251\bin>keytool -list -keystore server.keystore
输入密钥库口令:
密钥库类型: PKCS12
密钥库提供方: SUN

您的密钥库包含 0 个条目


C:\Program Files\Java\jdk1.8.0_251\bin>
```

### 修改证书条目的口令

修改server.keystore的口令

```
//刚刚被删除了，先导入server
keytool -importpass -alias server -keystore server.keystore
//修改成口令密码
keytool -keypasswd  -alias server -keystore server.keystore

C:\Program Files\Java\jdk1.8.0_251\bin>keytool -importpass -alias server -keystore server.keystore
输入密钥库口令:
输入要存储的口令:
再次输入口令:
输入 <server> 的密钥口令
        (如果和密钥库口令相同, 按回车):

C:\Program Files\Java\jdk1.8.0_251\bin>keytool -keypasswd  -alias server -keystore server.keystore
输入密钥库口令:
新<server> 的密钥口令:
重新输入新<server> 的密钥口令:

C:\Program Files\Java\jdk1.8.0_251\bin>
```



## SpringBoot跳转HTTPS

以下代码实现了从80端口跳转到了HTTPS

```
@Configuration
public class HttpsConfig {
    /**
     * 配置 http(80) -> 强制跳转到 https(443)
     */
    @Bean
    public Connector connector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(80);
        connector.setSecure(false);
        connector.setRedirectPort(443);
        return connector;
    }

    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory(Connector connector) {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(connector);
        return tomcat;
    }
}
```

## SpringBoot同时支持Http和Https

有俩种版本，1是让应用自动支持http，然后手动配置对Https的支持，2是配置自动支持https，如上面，然后让自动支持http

第二种实现简单如下

对于SpringBoot2来说

```
	@Bean
	public ServletWebServerFactory servletContainer() {
		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
		tomcat.addAdditionalTomcatConnectors(createStandardConnector());
		return tomcat;
	}
	
	// 配置http
	private Connector createStandardConnector() {
		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
		connector.setPort(port);
		return connector;
	}
```

SpringBoot1.5.X以下

```
    @Bean
    public EmbeddedServletContainerFactory servletContainer() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
        tomcat.addAdditionalTomcatConnectors(createStandardConnector()); // 添加http
        return tomcat;
    }
```