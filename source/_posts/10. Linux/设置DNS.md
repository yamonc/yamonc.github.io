---
title: 设置DNS
date: 2023-08-30 16:02:38
tags: [工具]
categories: [Linux]
recommend: false
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301601024.jpeg
comment: 是
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# 如何设置DNS

![the sun is setting over a desert landscape](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301601024.jpeg)

A服务器：192.168.110.1
B服务器：192.168.110.2

*使用A服务器作为源服务器
*使用B服务器作为目标服务器

一、将A服务器中的文件、文件夹传递到B服务器
1、文件
scp  /home/test/file.pdf root@192.168.110.2:/home/test 
[文件重命名]
scp  /home/test/1.pdf root@192.168.110.2:/home/test/2.pdf

2、文件夹（文件夹及其文件夹下的文件）
scp -r /home/test/files root@192.168.110.2:/home/test 
[不包含文件夹本身，文件中包含文件夹，子文件夹不能被传递]
scp  /home/test/files/* root@192.168.110.2:/home/test 

执行命令后提示输入目标服务器密码，输入密码即可。


二、在B 服务中将A服务器中的文件、文件夹传输过来
1、文件
scp root@192.168.110.1:/home/test/1.pdf /home/test
[文件重命名]
scp root@192.168.110.1:/home/test/1.pdf /home/test/2.pdf

2、文件夹
scp -r root@192.168.110.1:/home/test/files /home/test
[不包含文件夹本身，文件中包含文件夹，子文件夹不能被传递]
scp  root@192.168.110.1:/home/test/files/* /home/test

执行命令后提示输入源服务器密码，输入密码即可。


注意问题：
输入密码提示:Permission denied, please try again.
可能原因：
1、目标服务器/源服务器密码输入错误；
2、目标服务器目录权限问题，不允许写入；
    chmod 777  /home/test
3、ssh配置
    vi /etc/ssh/sshd_config 
    修改 PermitRootLogin 为 yes，保存
    重启服务： service sshd restart
    出现:Redirecting to /bin/systemctl restart sshd.service
    使用以下命令
        1、启动：systemctl start sshd.service
        2、重启：systemctl restart sshd.service
