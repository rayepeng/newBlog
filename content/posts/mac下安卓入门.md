---
title: "Mac下安卓入门"
date: 2020-11-28T17:28:16+08:00
draft: false
tags: ["安卓"]
categories: ["逆向"]
---



adb连接mumu模拟器



```
adb kill-server && adb server && adb shell
```



## mac远程调试mumu模拟器



```bash
# push ida的dbgsrc到模拟器中
adb push android_x86_server /data/local/tmp
chmod 777 android_x86_server
./android_x86_server

#端口转发
adb forward tcp:23946 tcp:23946
```



打开一个ida的新页面



![](https://gitee.com/xinyongp/image/raw/master/20201204105138.png)



连接，选择要调试的包



