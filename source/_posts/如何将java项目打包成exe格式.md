---
title: 如何将JavaFX的java项目打包成exe格式
date: 2023-06-16 11:22:52
tags: [java,exe]
categories: 教程
recommend: true
locate: 天津
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261746693.jpeg
---
# 如何将JavaFX的java项目打包成exe格式

![a dark tunnel with a light at the end](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306261746693.jpeg)

## 背景

最近在博客的时候，因为用到了hexo，所以必须按照hexo的格式上传文章，这样网站在显示的时候才会完整的显示出来：

![image-20230615204813105](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306152048160.png)

但是，每次写一篇，就得手动在文章的头部添加一些这些信息，而且添加完了之后，还要重新执行hexo三剑客才能部署到github的page上，所以就想着能不能按照，写一个客户端，用java写一个能运行在windows平台上的代码。

以下都是基于代码写好了之后，详情请看这篇文章->>>>>>>

## maven打包

打包方式的话，可以参考文章末尾的第一篇文章。

本文使用方法一会报错。所以使用方法二。

使用的插件为maven-assembly-plugin，只需要在pom文件里面添加这段代码：

```java
 <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass>com.yamon.convert/com.yamon.convert.HelloApplication</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

参考文章中有说的需要查看META-INF文件夹下的MAINIFEST.MF,需要查看里面的引用是否完整，我的内容比较少，也没有手动加。但是如果你要是有别的依赖包的话，需要手动添加一下，具体参考这篇文章：[将idea中的JavaFX项目打包成可执行的exe应用](https://blog.csdn.net/qq_29428909/article/details/122103131)

重新导一下maven包，然后打开maven，点击：

![image-20230616103621652](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161036716.png)

完成打包：

![image-20230616103635913](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161036953.png)

## exe4j将jar包导出成exe格式

>前记：
>
>之前也用过的别的，强烈不推荐使用jsmooth，感觉都是上古时期的工具了。按照教程一步一步操作，始终打不开。后来切换成exe4j之后，才成功完成打包动作。另外，打包方式的不同也有可能导致包的失效（毕竟不是springboot项目，怎么打都可以）

先下载exe4j文件，比较快。下载好了之后，完成安装。



1. 欢迎页面不用看，直接点下一步：

![image-20230616104018147](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161040185.png)

2. 项目类型选择jar包 in exe模式：![image-20230616104119283](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161041330.png)
3. 应用配置，这个应该是导出的文件名称，然后导出的目录位置：![image-20230616104201987](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161042024.png)
4. 配置执行文件，需要将高级选项点开，选择32/64/arm![image-20230616104240329](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161042379.png)
5. ![image-20230616104318653](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161043712.png)
6. ![image-20230616104331961](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161043997.png)
7. Java invocation：如果有额外的vm参数，需要调参的话，在上面填写；下面点击右侧绿色的加号，将刚才导出的包添加进来：![image-20230616104446922](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161044962.png)主类选择项目中的主类即可。
8. 配置JRE，尽量和当前系统环境一致：![image-20230616104543316](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161045352.png)
9. 坑点：这里需要在环境变量里面添加变量EXE4J_JAVA_HOME，然后地址写jre的地址。要不会报参考中的第三个错误；如果你的jdk版本没有jre的话，可以参考这篇文章：[待补充]。![](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161048283.png)
10. 然后一路下一步就行了。
11. 最后试一下，看是否能启动应用：![image-20230616104849351](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202306161048396.png)
12. 如果能启动的话，表示成功，点击save as，将这个配置保存到本地。

## 参考

[将idea中的JavaFX项目打包成可执行的exe应用](https://blog.csdn.net/qq_29428909/article/details/122103131)

[JDK11，JDK12 使用EXE4J 无JRE解决办法](https://www.cnblogs.com/ococo/p/15875003.html)

[No JVM could be found on your system. Please define EXE4J_JAVA_HOME to point to an installed 64-bit](https://blog.csdn.net/jiong9412/article/details/126481322)
