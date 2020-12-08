---
title: "Pc微信消息防撤回"
date: 2020-12-02T16:43:45+08:00
draft: true
---

用另一个账号发送消息，然后撤回，此时可以看到xml字符串

![](https://gitee.com/xinyongp/image/raw/master/20201202164433.png)

用ollydbg附加微信进程，在地址 `11C44A60` 处下内存写入断点。重新发送消息然后撤回可以命中断点

![](https://gitee.com/xinyongp/image/raw/master/20201202164956.png)

可以看到这里会将Unicode循环移动到 `11C44A60` 对应的内存区域

