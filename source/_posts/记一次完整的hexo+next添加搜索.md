---
title: 记一次完整的hexo + next添加搜索
date: 2018-01-11 18:25:09
tags:
    - hexo
    - algolia
categories: 
	- hexo
---

## 确认主题版本

``` bash
#Theme version
5.1.2
```

（themes\next\_config.yml）

## 安装hexo algolia

在hexo根目录，进行Hexo Algolia的安装：

``` bash
npm install --save hexo-algolia
```

若执行失败，请检查你的next版本

## algolia配置流程
通过[algolia官网](https://www.algolia.com/users/sign_up)注册，有github账号的可以直接登录。

![algolia_1.png](/upload/algolia_1.png)

登录成功进入控制台后，点击右上角创建index

![algolia_2.png](/upload/algolia_2.png)

通过左侧菜单进入API Keys页面，记录下里面的信息，之后需要写到hexo的站点配置文件中

![algolia_3.png](/upload/algolia_3.png)

选择ALL API KEYS，给当前Key追加权限，如下图所示：

![algolia_4.png](/upload/algolia_4.png)

![algolia_5.png](/upload/algolia_5.png)

（注意这一点，目前大部分教程都缺少这部分）

<!--more-->

在hexo根目录编辑_config.yml加入如下配置，具体key值参照上图填上自己的

``` bash
# Algolia Search
algolia:
  applicationID: 'Application ID'
  apiKey: 'apikey'
  adminApiKey: 'adminApiKey'
  indexName: 'yourIndeName'
  chunkSize: 5000
```

在git bash中执行hexo algolia，出现下图信息则表示上传成功

![algolia_7.png](/upload/algolia_7.png)

若出现如下提示，表示需要多执行一步操作

![algolia_8.png](/upload/algolia_8.png)

实际配置的时候也在这里卡了一下，网上的教程上对于这这部分都没有详细解释，最后还是参照官方说明

[hexo-algolia官方说明](https://www.npmjs.com/package/hexo-algolia#security-concerns)

原来新版本的hexo-algolia为了安全性，必须先配置一个environment variable，如下图所示：

![algolia_11.png](/upload/algolia_11.png)

多执行一步export HEXO_ALGOLIA_INDEXING_KEY='your Search-Only API Key'后，重新hexo algolia即可

![algolia_9.png](/upload/algolia_9.png)

上传成功后查看algolia上对应index，也会显示出相应信息

![algolia_6.png](/upload/algolia_6.png)

之后在\themes\next下找到_config.yml，修改配置如下：

``` bash
# Algolia Search
algolia_search:
  enable: true
  hits:
    per_page: 10
  labels:
    input_placeholder: 输入关键词进行搜索
    hits_empty: "找不到关于” ${query} ”的文章"
    hits_stats: "共找到 ${hits} 篇文章，花了 ${time} ms"
```

重新发布hexo g --deploy，再看看已经可以进行搜索了

![algolia_10.png](/upload/algolia_10.png)