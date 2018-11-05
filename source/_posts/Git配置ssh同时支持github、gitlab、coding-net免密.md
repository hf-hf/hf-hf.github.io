---
title: Git配置ssh同时支持github、gitlab、coding.net免密
date: 2018-11-05 14:30:52
tags:
    - git
---
![homePage](/upload/homePage/20181105161104.jpg)
<!--more-->

## 情景
实际工作中，我们经常会使用Github、Gitlab或者Coding.net其中的一种或多种，像我在开发中就同时使用了这三者，其中Github是个人的，Gitlab是公司内网服务器搭建的、Coding.net是为了解决Github屏蔽百度爬虫与其同步提交的，因此需要同时给这三者都配置ssh免密。

## 解决方法
之前配置单一的ssh时，我们会使用ssh-keygen -t rsa -C "yourEmail"来生成ssh的公私钥，生成中的配置都为默认，即直接点空格即可。
但是现在我们会对应Github、Gitlab和Coding.net生成三对公私钥，因此在生成时不能连敲三个空格，在第一个输入选项要输入生成密钥的名称，我这里分别创建了三对密钥，名称依次为：github、gitlab和coding，生成之后我们到用户下的.ssh目录。

可以看到其下有如下文件：

```
~/.ssh
$ ls
coding_id_rsa      github_rsa         gitlab_rsa
coding_id_rsa.pub  github_rsa.pub     gitlab_rsa.pub
```

现在我们在该目录下面再创建一个config文件，其作用是为了分配不同平台匹配不同的密钥，内容如下：

```
cat config

# GitHub.com server
Host github.com
HostName github.com
IdentityFile ~/.ssh/github_rsa

# Company GitLab server
Host 192.168.0.196
HostName 192.168.0.196
IdentityFile ~/.ssh/gitlab_rsa

# Coding.net
Host git.coding.net
HostName git.coding.net
IdentityFile ~/.ssh/coding_id_rsa
```

之后再将对应的公钥拷贝到指定服务的官网ssh配置中即可，现在使用Github、Gitlab和Coding.net都可以ssh免密了。