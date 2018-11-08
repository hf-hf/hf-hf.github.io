---
title: hexo配置使用多个deploy
date: 2018-07-14 13:53:15
tags:
    - hexo
    - deploy
categories: 
	- hexo
---

## hexo配置使用多个deploy

### 配置说明
#### 使用单个deploy，_config.yml：

```
deploy:
  type: git
  repo: git@github.com:hf-hf/hf-hf.github.io.git
  branch: master
```

#### 同时使用多个deploy，type需要在前面加"-"

```
deploy:
- type: git
  repo: git@github.com:hf-hf/hf-hf.github.io.git
  branch: master
- type: git
  repo: git@git.coding.net:hf-hf/hf-hf.github.io.git
  branch: hexo
```

### 百度链接自动提交
#### 使用baidu_url_submitter插件，需要在原deploy之外再添加一个新的deploy的类型，配置如下：

```
deploy:
- type: git
  repo: git@github.com:hf-hf/hf-hf.github.io.git
  branch: master
- type: baidu_url_submitter
```

#### 配置后，运行hexo delpoy后即会多执行新加的deploy baidu_url_submitter
