---
title: hexo解决github pages屏蔽百度爬虫的问题
date: 2018-09-11 09:47:18
tags:
    - hexo
categories: 
	- hexo
---

![homePage](/upload/homePage/20180911104701.jpg)
<!--more-->

## 情景
hexo博客提交百度收录，在百度站长工具中提示如下信息：

```
HTTP/1.0 403 Forbidden
Cache-Control: no-cache
Connection: close
Content-Type: text/html
```

响应码为403，很明显为授权被拒绝，在网上搜索了一下，发现其原因为github pages屏蔽了百度的爬虫(主要是屏蔽了爬虫的UA Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html ))，另外博主也介绍了两种解决方案：<em>CDN回源</em>和<em>为百度spider指定特殊线路</em>，详情介绍见[hexo定制and优化](https://kikoroc.com/2016/05/05/hexo-customize-and-optimization.html)。

## 方案选定
最终我选定了第二套方案，也就是为百度爬虫添加另一条线路，使用coding.net托管代码，coding.net并没有屏蔽百度爬虫，但是coding.net到底不如github来得稳定，所以原github上的分支也不会移除，博文将同时提交github和coding.net。

## 什么是coding？
简单的来讲，coding就是一个类似于github的开源代码仓库，其优点有全中文界面、可免费创建私有仓库、访问速度比github快等等，另外它也有一些有助于项目管理的其他辅助功能，部分地方使用起来相比github还要更加便捷一些。

## 方案实施

### hexo配置
修改hexo根目录的_config.yml，修改deploy配置同时提交github和coding.net，repo下添加两条键值对，记得key冒号之后一定要添加空格，yml的格式要求还是很严格的。

```
deploy:
- type: git
  repo: 
    github: git@github.com:hf-hf/hf-hf.github.io.git
    coding: https://git.coding.net/hf-hf/hf-hf.git
  branch: master
- type: baidu_url_submitter #此处为配置的自动提交百度爬虫
```

### coding.net配置
coding.net和github配置基本相同，成功注册以后将hexo部署到代码仓库（因为我之前的已经开启过了，这边我就用新的仓库的截图，实际配置时使用的仓库名为hf-hf）,然后需要开启一下Pages服务，见下图：

![github_pages_1.png](/upload/github-pages/github_pages_1.png)

开启成功后，页面就提供了Pages应用的访问地址。选择右上角的设置，可绑定自己的域名，这边我配置了首选是@解析，www开头的会跳转至@。

![github_pages_2.png](/upload/github-pages/github_pages_2.png)

PS：这里绑定域名需要coding.net会员等级为银牌以上（包含银牌），但是目前提供了绑定腾讯云免费升级的活动^^。

![github_pages_3.png](/upload/github-pages/github_pages_3.png)

### 域名解析配置

修改域名解析，将原github的解析路线修改为境外，新增解析CNAME到你的coding.net Pages应用的访问地址，我这边的访问地址是hf-hf.coding.me。

![github_pages_4.png](/upload/github-pages/github_pages_4.png)

这样从境外节点访问到的是你原github的地址，国内访问的就是coding.net了，在国内coding.net比github的访问速度还是要快一点的。

这样ping一下域名看一下修改后访问速度，可以看到访问的节点已经变成了hf-hf.coding.me。

![github_pages_5.png](/upload/github-pages/github_pages_5.png)

再试一下原来github的地址，可以明显看到访问coding.net相比github减少了50ms左右的响应时间。

![github_pages_6.png](/upload/github-pages/github_pages_6.png)

这样百度爬虫就可以爬取coding.net的博客啦，文章能够正常的被百度收录\_(:зゝ∠)\_ 。
