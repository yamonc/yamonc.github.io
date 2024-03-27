---
title: ---
date: 2024-03-21 10:36:16
tags: [222,222]
categories: [231321]
recommend: false
locate: 3213
cover: 21321
---

---
title: ---
date: 2024-03-21 10:35:37
tags: [222,222]
categories: [231321]
recommend: false
locate: 3213
cover: 21321
---

---
title: jdk17没有JRE文件夹的解决方法
date: 2024-03-21 10:25:36
tags: [222,222]
categories: [231321]
recommend: false
locate: 3213
cover: 21321
---

# jdk17没有JRE文件夹的解决方法

找到JDK的安装目录，然后在该目录下输入以下指令即可生成

```bash
bin\jlink.exe --module-path jmods --add-modules java.desktop --output jre
```

## 参考

[Windows系统安装JDK17没有jre文件夹解决方法](https://blog.csdn.net/u012993896/article/details/123150376)
