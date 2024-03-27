---
title: 从0到1部署博客系统到Github Page（0成本）
date: 2023-06-13 14:32:07
tags: [github,博客,hexo]
categories: 教程
description: 近来的一些随想。
recommend: true
locate: 天津
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261745323.jpeg
---

# 从0到1部署博客系统到GitHub Page（0成本）

![a view of a mountain range with a pink sky in the background](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261745323.jpeg)

先po一下博客的地址：

[热爱并分享生活](https://yamonc.github.io/)

主要选型：hexo+[Acrylic](https://next-docs.acrylic.org.cn/configuration/comment)+Github Page+[twikoo](https://twikoo.js.org/quick-start.html#vercel-%E9%83%A8%E7%BD%B2)（评论）+[Vercel](https://vercel.com/dashboard)（自动化部署twikoo）+ [mongodb cloud](https://cloud.mongodb.com/v2/64870703b548175c8b6e3255#/clusters)

## 背景

从去年年末到今年五月底、六月初，一直在想，重新搞一个博客网站玩玩，正好也想学习一下vue的前端知识。然后就上网上找了一下模板，刚开始使用的是vue-element-admin，但是无奈该模板是后台管理系统，所以又找人给搭了一个特别简单的网站。在接下来的几个月的时间内，吭哧吭哧地干，一点点的干，从前端的博客列表、标签分栏列表、各种crud页面的编写，期间一度的怀疑自己在干什么，感觉没什么意义，但最后还是做出来了点效果，但是样式过于丑陋，始终不满。

终于，在上周五晚上，反思了下最近这半年正在做的事儿，感觉什么都做了，又感觉什么都没做。在想到博客的时候，自己明明想要的是能够做到文档归档功能就行，其他的多余的什么登录、延迟删除博客、撤回删除博客有什么用呢？这不就是无用功了吗？

说回来，之前就看到一些博主利用github来写一些文档，例如一些网页后面的github.io的后缀。

目前常见的选型有这几种：

1. 自己搭建的博客（成本高，需要自己买服务器，时间成本也高，但是自定义高，可玩性高）
2. [vuepress](https://vuepress.vuejs.org/zh/)，利用vuepress纯前端静态资源部署CMS内容。
3. hexo，模板较多，类似vuepress（本文选用的是这个）

关于博客系统的选型，之前就知道有hexo，做博客期间的这半年，还想要不将hexo搬到服务器上，在服务器上部署一套就行了。后来无意中了解到，gitee和github可以提供服务器资源，提供免费站点（gitee也可以，但是需要提供身份证认证）。

## 搭建使用的组件

首先，需要使用git，用来将代码push到github上。

node.js和npm包管理工具用来安装前端必须的包；

hexo，博客模板网站。

这几个之间的关系是：

node.js和npm为底层支撑，用来支撑项目的启动；

hexo为样式渲染组件，hexo通过node.js和npm完成部署。

1. [node.js](https://nodejs.org/en),下载直接安装即可。
2. [git](https://git-scm.com/)，同上，安装次版本即可。
3. npm， 包管理工具。
4. [hexo](https://hexo.io/zh-cn/docs/)，博客模板网站

## 本地部署

开始部署工作，默认已经配置了git、npm和nodejs。

### 安装hexo

[参考官方文档](https://hexo.io/zh-cn/docs/#%E5%AE%89%E8%A3%85-Hexo)

```bash
npm install -g hexo-cli
```

或者安装全量包：

```bash
npm install hexo
```

### 初始化hexo文件夹

安装好了之后，需要在你的pc上，找一个地儿，创建一个blog的文件夹：

比如我选择的地方：

![image-20230613114643441](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131146514.png)

找好了之后，右键空白处，选择git bash here

首先声明一下，这个文件夹是hexo的：

```bash
hexo init .
```

### 安装依赖

接着，安装下依赖：

```bash
 npm install
```

安装好了依赖之后，可以查看目录结构：

```text
 .
 ├── _config.yml # 网站配置信息
 ├── package.json # 应用程序信息
 ├── scaffolds # 模板文件夹
 ├── source # 存放用户资源
 |   ├── _drafts
 |   └── _posts
 └── themes # 主题文件夹
```

### 验证

输入hexo指令：

```text
 # 新建博客
 hexo n "test"
 # 生成静态网页
 hexo g
 # 打开本地服务器
 hexo s
```

查看控制台是否有报错，如果有的话，查看原因。

（ps：我记得，在这里好像有问题，但是百度绝对能百度出来，原因大概是少装了依赖，再装下依赖就不会报错了）

这个时候，控制台应该一个localhost的连接，访问连接即可。

![image-20230613135632039](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131356094.png)

（已经更换过主题的，所以你的有可能跟我不一样，可以从简单开始，先将最简单的部署到Github的Page站点）

## 部署到Github Page站点

访问[Page文档](https://docs.github.com/zh/pages/getting-started-with-github-pages)了解更多。

首先，在Github上创建一个仓库，名字一定要和你的用户名一样：

![image-20230613140138288](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131401328.png)

可以添加一个README的markdown文件，方便pull（仓库为空的话，不能pull），或者更简单的，不需要你创建这个仓库，在hexo的目录下，使用以下命令，将hexo的目录push到github上，但是名字要改成上面这个图一样的（用户名.github.io，我这里是将hexo和github本地仓库分开的）。

这时候，有可能会出现github认证问题，需要你输入github的账号密码，虽然你输入的是正确的，但是总是pull/push不成功。

解决方案：升级你的git版本。

创建之后，创建一个文件夹，然后pull下来远端的代码：

![image-20230613140821833](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131408878.png)

注意，在**博客的根目录**下，找到hexo的主配置（定义：其他主题都为次配置）：

![image-20230613141118667](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131411707.png)

配置主配置：

可配置的内容：

```xml
# Site
title: yamon，分享并热爱生活 # 浏览器tab栏上显示内容
subtitle: '' # 可以写
description: 'a blog 一个分享技术与生活的博客' # 应该是百度搜索出来的时候用的
keywords: Rootlex, blog, 技术, 博客 # 搜索关键字
author: yamon # 作者，footer用的应该
language: zh-CN # 语言
timezone: '' # 时区

url: http://yamonc.github.io # 分享的时候用的url
permalink: :year/:month/:day/:title/ # 应该是点开文章后，url上显示的格式
permalink_defaults:
```

重点内容来了：

```

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/ 
## 这个是主题配置，如果更换主题的话，需要在这里修改下名字，具体后续会讲
theme: Acrylic

# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
# 部署在github上，type是部署方式；repo是部署的仓库，就是你要把代码push到哪个仓库上；branch：部署的分支。
deploy:
  type: git
  repo: https://github.com/yamonc/yamonc.github.io
  branch: main
```

保存之后，再执行三或四个指令，这四个指令会贯穿到从始至终：

```xml
hexo clean # 清除缓存
hexo g # 生成静态文件
hexo s # 启动服务（自己debug用，或者本地查看一下会不会报错）
hexo d # 开启部署
```

接着，点到用户名.github.io的仓库里面，找到setting：

![image-20230613142035115](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131420206.png)

找到左侧的pages选项：

如果出现了这行字，表示部署成功，可以通过这个域名访问：

![image-20230613142150188](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131421243.png)

> 如果没有部署成功的话，可以查看一下这里的内容，是否配置成功，主要是：
>
> 1. build and deployment选项的branch分支是否选择正确，是否是main分支，一般都是main分支。
> 2. 在本地hexo目录下，使用hexo s本地部署，查看控制台是否报错，有可能是博客的front markdown没有写好，格式不对，多加了空格什么的。
> 3. 或者不行的话，到仓库的Action选项卡中，重新跑一遍job试一下。
>
> ![image-20230613142606737](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131426822.png)
>
> 另外，这里还有一个Custom domain自定义域名，如果有域名的话，可以在这里配置，具体配置的话，另起一篇再讲吧。

最后部署成功的样子：

![image-20230613142639000](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131426206.png)

## 参考

[超详细 Hexo + Github Pages 博客搭建教程](https://zhuanlan.zhihu.com/p/370635512)



