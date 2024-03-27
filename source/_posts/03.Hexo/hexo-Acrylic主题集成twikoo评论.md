---
title: hexo-Acrylic主题集成twikoo评论
date: 2023-06-13 17:51:07
tags: [twikoo,博客,hexo]
categories: 教程
description: 近来的一些随想。
recommend: true
locate: 天津
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261745868.jpeg
---

# hexo-Acrylic主题集成twikoo评论

![a large building with a clock tower in the middle of it](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261745868.jpeg)

虽然hexo使用了Acrylic的主题，但是评论模块一直没有打开，官方给出的文档也没有相关的文档，所以上网上找到了一篇参考，做了一些详细的配置。

主要完成的是评论模块的功能实现，增加、提醒等功能。具体可以查看twikoo[文档](https://twikoo.js.org/)

twikoo文档上给出了很多部署的方式，主要分两类，一个是云函数的部署，另外一种是前端部署。

![image-20230613171408109](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131714157.png)

这里使用的是Vercel部署。

## Vercel部署

主要的部署流程：

1. 申请mongodb账号
2. 创建免费的mongodb数据库，区域的话，可以选择hongkang。
3. 查看配置信息，在clusters页面上点击connect，然后选择Driver，复制数据库信息。
4. 申请Vercel账号。
5. 将twikoo一键部署到Vercel上
6. 在Vercel上，找到Setting-Environment Variable，添加MONGODB_URI，值为第3步的数据库信息。
7. 进入 Deployments , 然后在任意一项后面点击更多（三个点） , 然后点击Redeploy , 最后点击下面的Redeploy。
8. 进入 Overview，点击 Domains 下方的链接，如果环境配置正确，可以看到 “Twikoo 云函数运行正常” 的提示。
9. Vercel Domains（包含 `https://` 前缀，例如 `https://xxx.vercel.app`）即为您的环境 id。

### 主要流程

#### mongodb上的操作

正常流程注册就行，应该可以使用github注册的。注册OK了之后，会有引导，选择最右侧免费的cluster（土豪忽略这个），然后进行配置：

![创建页面](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131721646.png)

这里我没有选择官方推荐的，我选择的是距离咱们最近的Hong Kong。然后创建数据库即可。

接下来，需要配置mongodb数据库，设置ip为0.0.0.0/0即可：

![配置ip.png](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131723940.png)

创建数据库用户，这里的paasword后面有用到：

![配置用户.png](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131723680.png)

返回到cluster界面，点击connect：

![CONNECT1.png](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131723711.png)

这里跟参考里面的不一样，这里要选择device还是什么来着，也是application，只不过描述不一样。

点进去之后，除了记录一下数据库的信息之外，还需要在hexo目录下，npm装一下mongodb的包，要不twikoo还是不能用。

#### 部署到Vercel

申请Vercel账号，需要手机号认证，去下拉框里面找咱们国家就行，输入验证码之后就注册成功了。

开始部署：

https://vercel.com/import/project?template=https://github.com/imaegoo/twikoo/tree/main/src/server/vercel-min

点击上述连接。

Repository Name这里可以随便输入，建议输入twikoo，给自己的github上创建一个twikoo的仓库。

点击Create进入下一步。

等Import之后，点击visit进入仓库。

这时候，应该会报错，还需要配置环境变量。

##### 新建环境变量

进入`Settings - Environment Variables`页面。
新建一个NAME为`MONGODB_URI`;VALUE为你在前面记录到的`数据库连接字符串`的环境变量。

![Setting.png](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131728847.png)

点击Deployment重新部署一下：

![image-20230613173028453](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131730574.png)

部署如果成功的话：

![image-20230613173101471](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131731529.png)

Domain下面的连接就是要配置的envId。

至此的话，如何你点击Domain下面的连接可以正常访问的话，那可以跳过以下几步，直接到邮箱配置这块了，但是如果无法访问的话，需要配置一下二级域名才能完成访问的。

到阿里云控制台上（你域名的服务商），找到域名：

![image-20230613173646350](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131736438.png)

点击添加域名：

添加子域名：

![img](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131737855.webp)

提示TXT授权校验：

将TXT授权校验的内容保存好之后，对主域名进行添加子域名解析：

添加解析记录：

![img](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131738760.webp)

添加好了之后，再返回点击验证。

最后将二级域名解析到Vercel中：

在二级域名中添加两条记录（不用修改）：

![image-20230613173948054](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131739108.png)

| 主机记录 | 记录类型 | 解析请求来源(isp) |        记录值        |   TTL   |
| :------: | :------: | :---------------: | :------------------: | :-----: |
|  twikoo  |  CNAME   |       默认        | cname.vercel-dns.com | 10 分钟 |
|    @     |    A     |       默认        |     76.76.21.21      | 10 分钟 |

在Vercel项目中添加Domains域名

点击Project Settings-》Domains，添加域名即可

![image-20230613174240816](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131742880.png)

替换之前的envid：

![image-20230613174157503](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131741532.png)

#### 实现邮件通知

以qq邮箱为例：

需要得到邮箱的pop3/SMTP的凭证，现在已经改版了，不太好找了。可以这么操作：

登录上邮箱之后，选择设置-》账户，找到POP3一栏。

![image-20230613174604011](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131746070.png)

点击如何设置？

跳转到这个页面：

![image-20230613174626026](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131746090.png)

点它就行了，然后一步一步配置就行，该发短信就发短信。

得到凭证之后，开始配置邮箱提醒：

SENDER_EMAIL: <你的QQ邮箱地址>
SMTP_SERVICE: <你的邮件服务提供商>
SMTP_HOST: <自定义 SMTP 服务器地址>
SMTP_PORT: <自定义 SMTP 端口>
SMTP_SECURE: <自定义 SMTP 是否使用 TLS>
SMTP_USER: <邮件通知邮箱用户名>(需与SENDER_EMAIL一致)
SMTP_PASS: <邮件通知邮箱密码>(授权码)

一般在评论左下角会有一个小齿轮，点击进去，第一次需要注册，注册完了之后按照这个配置一下就行：

点击配置管理-》邮件通知，按照以下进行配置：



![配置2.png](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131748658.png)

![配置1.png](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306131748532.png)

然后点击最下面的邮件通知测试，完成测试。

## 参考

[部署Twikoo评论系统及其邮件推送(Vercel)](https://blog.csdn.net/weixin_58068682/article/details/122770936)

[关于Vercel被墙导致获取Twikoo评论失败的解决方案](https://www.jianshu.com/p/02fb996c4638)