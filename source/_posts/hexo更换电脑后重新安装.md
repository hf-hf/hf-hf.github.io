---
title: hexo更换电脑后重新安装
date: 2018-11-05 14:12:31
tags:
    - hexo
---

![homePage](/upload/homePage/20181105161102.jpg)
<!--more-->

## 情景
之前已经将hexo博客的源码提交到了Git部署仓库的hexo分支，这次更换电脑后需要重新安装、编译，现将每一步都记录一下。

## 下载源码
将之前提交到Git的hexo源码下载到本地。

```
$ git clone https://github.com/hf-hf/hf-hf.github.io.git
Cloning into 'hf-hf.github.io'...
remote: Enumerating objects: 1818, done.
remote: Counting objects: 100% (1818/1818), done.
remote: Compressing objects: 100% (450/450), done.
remote: Total 8994 (delta 809), reused 1675 (delta 684), pack-reused 7176
Receiving objects: 100% (8994/8994), 8.46 MiB | 10.00 KiB/s, done.
Resolving deltas: 100% (3756/3756), done.
Checking connectivity... done.
```

下载完毕之后，切换分支到hexo。

```
/d/hexoTmp/hf-hf.github.io (master)
$ git checkout hexo
Branch hexo set up to track remote branch hexo from origin.
Switched to a new branch 'hexo
```

该分支下的文件如下，其中hexo-install和hexo-deploy为我自己配置的发布脚本，其余文件为提交到Git的hexo源文件：

```
_config.yml  hexo-install  package-lock.json  scaffolds/  themes/
hexo-deploy  package.json  README.md          source/
```

## 安装hexo
在源文件目录右击打开Git，执行npm install hexo-cli -g，输出一下信息表示安装成功。

```
$ npm install hexo-cli -g
+ hexo-cli@1.1.0
added 126 packages, removed 3 packages, updated 28 packages and moved 1 package in 32.959s
```

其中可能会出现一些warn，我们暂时忽略它。

## 安装依赖
执行npm install，安装一下依赖插件，如果需要本地调试，要单独安装一下npm i hexo-server。

```
$ npm install

> nunjucks@3.1.3 postinstall D:\hexoTmp\hf-hf.github.io\node_modules\hexo\node_modules\nunjucks
> node postinstall-build.js src

npm WARN rollback Rolling back readable-stream@2.3.6 failed (this is probably harmless): EPERM: operation not permitted, lstat 'D:\hexoTmp\hf-hf.github.io\node_modules\fsevents\node_modules'
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.4 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.4: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

added 446 packages in 36.652s

$ npm i hexo-server
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.4 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.4: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ hexo-server@0.2.2
removed 3 packages and updated 13 packages in 10.336s
```

## 编译生成静态文件并访问
hexo generate编译生成静态文件，hexo server本地启动服务访问，输入http://localhost:4000/ 就可以看到页面了。

## hexo构建无法生成index-html等文件问题
如果你在上一步构建后，发现生成的文件中没有index.html，使用hexo server启动本地服务，访问后也是提示没有index.html。

那么你需要手动再安装一下未安装的插件，使用npm ls --depth 0查看缺失的插件。

```
$ npm ls --depth 0
hexo-site@0.0.0 D:\hexoTmp\hf-hf.github.io
+-- hexo@3.7.1
+-- hexo-algolia@1.2.4
+-- hexo-baidu-url-submit@0.0.5
+-- hexo-deployer-git@0.3.1
+-- hexo-generator-archive@0.1.5
+-- hexo-generator-baidu-sitemap@0.1.2
+-- hexo-generator-category@0.1.3
+-- hexo-generator-index@0.2.1
+-- hexo-generator-sitemap@1.2.0
+-- hexo-generator-tag@0.2.0
+-- hexo-renderer-ejs@0.3.1
+-- hexo-renderer-marked@0.3.2
+-- hexo-renderer-stylus@0.3.3
`-- hexo-server@0.2.2

npm ERR! missing: mkdirp@0.5.1, required by node-pre-gyp@0.10.0
npm ERR! missing: minimist@0.0.8, required by mkdirp@0.5.1
npm ERR! missing: minimatch@3.0.4, required by ignore-walk@3.0.1
//...
```

可以看到输出了很多npm ERR! missing npm ERR! XXX。

使用npm install XXXX --save，逐一安装缺失的插件。

```
$ npm install mkdirp --save
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.4 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.4: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ mkdirp@0.5.1
updated 1 package in 7.286s
//...
```

因为可能缺失的插件较多，这里介绍一个快捷的办法，直接拷贝命令行中输出的所有npm ERR!，拷贝到文本编辑器中，批量替换前缀修改为npm install，批量修改后缀为--save，然后直接都拷贝到Git命令行中即可，它会自动从上到下一条条安装。

待全部插件安装完毕后重新构建hexo，可以看到能够正常生成index.html文件了，hexo server访问也正常。

## 最后分享一下hexo一键部署脚本

```
$ cat hexo-deploy
hexo clean && hexo g --deploy
```

将该脚本保存到Git下的cmd目录，即可直接在命令行中使用文件名执行，若有其他插件需要部署时同时执行，也可继续添加&&后再进行追加，如&& hexo algolia。
