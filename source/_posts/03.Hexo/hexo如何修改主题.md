---
title: hexo如何修改主题？
date: 2023-06-13 15:46:07
tags: [博客,hexo]
categories: 教程
description: 近来的一些随想。
recommend: true
locate: 天津
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261746884.jpeg
---

# hexo如何修改中意的主题？

![an aerial view of a tree in a field](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261746884.jpeg)

推荐Acrylic主题，样式比较好看，文档也比较完善，但是也有踩坑。

![image-20230613150638064](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131506443.png)

不过，还是感谢[@hexo-theme-Acrylic](https://github.com/hexo-theme-Acrylic/Hexo-Theme-Acrylic-Next)的开源项目。

修改主题可以参考文档修改，[hexo acrylic](https://next-docs.acrylic.org.cn/)。

## 安装主题

注意在hexo的根目录下执行指令，用来安装主题：

```git
git clone -b main https://github.com/hexo-theme-Acrylic/Hexo-Theme-Acrylic-Next.git themes/Acrylic
```

## 应用主题

修改主配置文件config.yml,修改主题配置：

```json
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: Acrylic
```

## 使用主题配置

macos/linux:

```bash
cp -rf ./themes/Acrylic/_config.yml ./_config.Acrylic.yml
```

windows:

复制 `/themes/Acrylic/_config.yml` 此文件到 **Hexo** 根目录，并重命名为 `_config.Acrylic.yml`

## 更新主题

在hexo主文件夹下找到themes文件夹，进入之后找到Acrylic，并pull代码。

直接hexo clean、hexo g、hexo s；看是否有问题。

一般有的问题，都可以在百度上搜索，一般都是少装了npm包，npm install一下就可以了。

比如这个： npm install hexo-deployer-git --save

## Acrylic一些的必要配置

首先按照文档，配置一下Hexo配置文件：

```xml
# Site
title: RootlexBlog
subtitle: ''
description: 'a blog 一个分享技术与生活的博客'
keywords: Rootlex, blog, 技术, 博客
author: Rootlex
language: zh-CN # 主要是这里不一样，默认的是en
timezone: ''

# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: http://example.com # 这块也建议修改成自己github的仓库地址，比如xxx.github.io
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```

上述配置，需要注意两点：

1. language需要改成zh-CN；
2. url修改成为自己的配置。

### 主题配置

```
site:
  name:                   # 左上角显示内容
    class:  text          #i_class/text/img 
    custom: Acrylic       #自定义内容
  siteIcon: /img/avatar.webp # 网页icon
  icon: /img/avatar.webp     # 页脚的icon、加载动画默认的icon
  icp:                       # ICP

```

其他的内容按照文档配置就行。

## 其他注意的点

### 一、文章顶部搜索栏：search搜索

建议使用本地搜索，本地搜索的话，需要安装hexo-generator-searchdb插件。

然后在_config.yml添加以下配置：

```
search:
  path: search.xml
  field: post
  content: true
  format: html
```

修改_config.arcylic.yml

```
  local_search:           #本地搜索
    enable: true
```

### 二、 文章详情meta数据没有显示

参考[《meta疑问及front-matter参数疑问》](https://github.com/hexo-theme-Acrylic/Hexo-Theme-Acrylic-Next/issues/153)

补充几句吧：

需要安装hexo-wordcount包：

```bash
npm i --save hexo-wordcount
```

一定要在次配置，主题配置中找到wordcount，并且设置为true：

![image-20230613162023688](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131620729.png)

### 三、twikoo评论无法显示

需要部署twikoo服务才能实现评论，具体参考文档[twikoo文档](https://twikoo.js.org/)

或查看下一篇文章。
