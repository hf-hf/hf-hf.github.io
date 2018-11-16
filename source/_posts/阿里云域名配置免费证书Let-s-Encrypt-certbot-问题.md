---
title: 阿里云域名配置免费证书Let's Encrypt(certbot)问题
date: 2018-11-16 14:55:31
tags:
    - linux
    - cerbot
    - 阿里云
---
![homePage](/upload/homePage/20181116153002.jpg)
<!--more-->

## 情景
今天使用阿里云域名通过Let's Encrypt(certbot)申请ssl证书，参照[cerbot官网](https://certbot.eff.org/lets-encrypt/centos6-nginx)中的步骤，报出如下信息：

```
[root@instance-zgdnfxxv bak]# ./certbot-auto certonly -d *.hunfan.top -d hunfan.top --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
Saving debug log to /var/log/letsencrypt/letsencrypt.log
...
Failed authorization procedure. hunfan.top (dns-01): urn:ietf:params:acme:error:dns :: DNS problem: NXDOMAIN looking up TXT for _acme-challenge.hunfan.top, hunfan.top (dns-01): urn:ietf:params:acme:error:dns :: DNS problem: NXDOMAIN looking up TXT for _acme-challenge.hunfan.top
```

## 原因
调查后发现，该问题出现原因为cerbot的验证地址没有配置相应的解析规则，无法通过验证导致，并且只有在使用阿里云域名时才会出现，在申请时有如下信息。

![cerbot_1.png](/upload/cerbot/cerbot_1.png)

## 解决方案
需要将该地址在阿里云域名解析中添加一条TXT类型的解析，主机记录为上图中的A，解析的记录值为上图中的B，具体配置内容如下。

![cerbot_2.png](/upload/cerbot/cerbot_2.png)

添加保存后，回到服务器，继续剩下的申请步骤。

PS：域名解析是需要一段时间生效的，可通过该命令查询新增解析是否生效：dig -t txt _acme-challenge.yourdomain.com @8.8.8.8。

生成证书成功后，控制台会打印如下信息：
```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/hunfan.top/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/hunfan.top/privkey.pem
   Your cert will expire on 2019-02-14. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

证书生成在/etc/letsencrypt/live/对应域名的文件夹下，接下来只需要在nginx中添加https配置引用证书即可。
