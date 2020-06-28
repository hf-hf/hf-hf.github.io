---
title: 解决爬取Https页面报unable to find valid certification path to requested target的问题
date: 2020-06-28 10:53:51
tags:
    - https
    - 证书
---
![homePage](/upload/homePage/20200628154000.jpg)
<!--more-->

## 情景
最近调试爬虫程序，对[ershcimi](https://www.ershicimi.com/)进行爬取时报错，堆栈信息如下：

```
09:34:59.523 main  com.kindle.utils.CrawlerUtils getHtml error!
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
	at sun.security.ssl.SSLSocketImpl.fatal(SSLSocketImpl.java:1949)
	at sun.security.ssl.Handshaker.fatalSE(Handshaker.java:302)
	at sun.security.ssl.Handshaker.fatalSE(Handshaker.java:296)
	at sun.security.ssl.ClientHandshaker.serverCertificate(ClientHandshaker.java:1509)
	at sun.security.ssl.ClientHandshaker.processMessage(ClientHandshaker.java:216)
	at sun.security.ssl.Handshaker.processLoop(Handshaker.java:979)
	at sun.security.ssl.Handshaker.process_record(Handshaker.java:914)
	at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1062)
	at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1375)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1403)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1387)
	at sun.net.www.protocol.https.HttpsClient.afterConnect(HttpsClient.java:559)
	at sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:185)
	at sun.net.www.protocol.https.HttpsURLConnectionImpl.connect(HttpsURLConnectionImpl.java:153)
	...
Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:387)
	at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:292)
	at sun.security.validator.Validator.validate(Validator.java:260)
	at sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:324)
	at sun.security.ssl.X509TrustManagerImpl.checkTrusted(X509TrustManagerImpl.java:229)
	at sun.security.ssl.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:124)
	at sun.security.ssl.ClientHandshaker.serverCertificate(ClientHandshaker.java:1491)
	... 46 common frames omitted
Caused by: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at sun.security.provider.certpath.SunCertPathBuilder.build(SunCertPathBuilder.java:141)
	at sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:126)
	at java.security.cert.CertPathBuilder.build(CertPathBuilder.java:280)
	at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:382)
	... 52 common frames omitted
```

## 原因分析
扫了一眼报错的堆栈信息，很明显就能看到关键词ssl，因为爬取出错的也是Https的页面，所以基本可以推测出是关于站点证书的问题。

那么具体是什么问题导致出现该错误的呢，这里我们首先要说明一个概念：证书链。

> ### 什么是证书链？
> 浏览器是怎么保证访问的网站是正经的官方网站而不是其他的钓鱼网站呢，Chrome浏览器访问网站时，可信任的网站地址旁边会有一个绿色的锁标志，表明该网站是可信任的，它是怎么知道该网站是可信任的呢。
> 因为浏览器会内置一些证书，其他证书都是由这些证书签发的，通过内置的证书来验证其他证书的有效性。这些浏览器内置的证书叫做Root CA(根CA证书)，其他网站的证书都是由Root CA证书一层一层往下签发的。

我们可以点击Chrome浏览器网站地址旁边的锁标志，选择证书点击证书路径查看网站的证书链，如下图：
![unable-to-find-valid-certification-path-to-requested-target_1](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_1.png)

从上到下依次是根证书、中级证书、网站证书，中级证书可能会有多级。根证书是终端设备预装信任的，所以是不需要配置的。以Windows系统为例，Google的Chrome浏览器使用的是操作系统预装的证书库，我们在Windows运行中输入certmgr.msc即可打开证书管理器，见下图。
![unable-to-find-valid-certification-path-to-requested-target_2](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_2.png)

我们现在知道根证书是设备级信任的，那么中级证书有什么作用呢？

> ### 什么是中级证书？
> 中级证书相当于是我们的根证书的替身。我们之所以使用中级证书，是因为我们必须在根证书上建立许多安全层，从而确保根证书的密钥绝对不会被任何人访问。
> 不过，由于根证书自身签署了中级证书，因此中级证书就可以用于签署我们的客户安装的SSL 并维持“信任链”。

中级证书是保证根证书安全性的，我们的网站向中级证书颁发者申请的证书，也就是证书路径最下级的网站证书。如果我们的网站证书一旦过期，在通过Https访问网站时浏览器立马会检测到并进行阻断，提示“这不是私密链接”等风险提示。

> ### 证书认证原理
> 服务器首先生成一个密钥对，把公钥提交给CA
> CA用自己的私钥对服务器提供的公钥进行签名得到证书
> Https服务器在与客户端进行连接的时候会将证书和公钥一起发给客户端，客户端用CA的公钥对证书进行验证，对比一致则证明该证书确实是CA发布的。
  
关于证书的认证原理见上方说明，我们现在知道了证书的信任链，那么现在试着在证书管理器中找一下图1中的根证书颁发者DST Root CA X3和中间证书颁发者Let's Encrypt Authority X3，见下图3、4：
![unable-to-find-valid-certification-path-to-requested-target_3](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_3.png)

![unable-to-find-valid-certification-path-to-requested-target_4](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_4.png)

能够看到在受信任的根证书颁发机构下存在DST Root CA X3，在中间证书颁发机构下存在Let's Encrypt Authority X3，且都是有效的证书没有过期，那么回到我们的问题，现在可以得出结论了关于报错提示站点证书的问题，就是因为我们本地缺少网站证书www.ershicimi.com。

这里所谓的本地，即我们的代码运行的环境，如我这里使用的是Java，那么对应的就是jdk中的证书库缺少该网站证书。

## 解决方案
既然根源的问题已经找到了，那么现在有两种解决问题的思路。

### 信任所有的Https证书
第一种的思路是比较暴力，虽然我现在只是网站证书缺失，但是我可以直接跳过所有Https证书的验证，意思就是我信任所有的Https证书，直接在代码发起请求前多加一段信任所有Https的证书，代码如下：

```
private static void trustAllHttpsCertificates() throws Exception {
    javax.net.ssl.TrustManager[] trustAllCerts = new javax.net.ssl.TrustManager[1];
    javax.net.ssl.TrustManager tm = new miTM();
    trustAllCerts[0] = tm;
    javax.net.ssl.SSLContext sc = javax.net.ssl.SSLContext.getInstance("SSL","SunJSSE");
    sc.init(null, trustAllCerts, null);
    javax.net.ssl.HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
}
```

第一种方案比较简单，网上的示例代码也很多，这里就不再赘述了，但是这种方法显然是不安全的。

### 配置缺失的证书

第二种则是既然缺少网站证书，那么就把缺失的证书导入进去，在Java环境中，就是把证书导入到jdk的证书列表中。

#### 网站证书下载

首先我们需要在浏览器中手动下载缺少的网站证书。

![unable-to-find-valid-certification-path-to-requested-target_5](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_5.png)

在详细信息中点击复制到文件。

![unable-to-find-valid-certification-path-to-requested-target_6](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_6.png)

进入证书导出向导，下一步后选择导出格式Base64编码，命名为ershicimi.cer保存到本地。

![unable-to-find-valid-certification-path-to-requested-target_7](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_7.png)

#### 网站证书导入

下载完缺失的证书后，我们需要看一下项目内使用的JDK路径。

![unable-to-find-valid-certification-path-to-requested-target_8](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_8.png)

进入该路径的bin文件夹下，以管理员权限运行cmd，执行以下命令导入证书：
```
keytool -import -file ershicimi.cer -keystore "D:\Java\jdk1.8.0_60\jre\lib\security\cacerts" -alias ershicimi
# 执行后提示输入密钥库口令，Java中cacerts证书库默认密码为changeit，我们输入changeit，回车
输入密钥库口令：changeit
...
# 此时会输出一串的证书信息，并提示是否信任此证书，我们输入Y，回车
是否信任此证书?[否]: Y
证书已添加到密钥库中
```

至此缺失的网站证书已导入到JDK下的JRE证书库cacerts中，如果遇到报错java.io.FileNotFoundException: ershicimi.cer，请确认-file后需要指定要导入证书的路径。

此时再运行程序，请求Https地址无报错，返回正常。

![unable-to-find-valid-certification-path-to-requested-target_9](/upload/ssl/unable-to-find-valid-certification-path-to-requested-target_9.png)

### keytool指令扩展
如果你不小心导入错误了想要删除之前导入的证书，或者想要查看新导入的证书，你可以执行以下的命令：

```
# 查看cacerts中的证书列表
keytool -list -keystore "%JAVA_HOME%/jre/lib/security/cacerts" -storepass changeit

# 删除cacerts中指定名称的证书
keytool -delete -alias ershicimi -keystore "%JAVA_HOME%/jre/lib/security/cacerts" -storepass changeit

# 导入指定证书到cacerts
keytool -import -alias ershicimi -file ershicimi.cer -keystore"%JAVA_HOME%/jre/lib/security/cacerts"
```