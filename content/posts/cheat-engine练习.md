---
title: "Cheat Engine练习"
date: 2020-12-07T09:56:54+08:00
draft: false
tags: ["CE"]
categories: ["逆向"]
---



## 步骤2

![](https://gitee.com/xinyongp/image/raw/master/20201207095923.png)

搜索修改即可

![](https://gitee.com/xinyongp/image/raw/master/20201207095955.png)



## 步骤3



![](https://gitee.com/xinyongp/image/raw/master/20201207100252.png)

这里首先搜索未初始化的值，然后点击打我

数值会减少，搜索数值减少了

![](https://gitee.com/xinyongp/image/raw/master/20201207100344.png)

又已知这个数值在0到500之间，修改为5000

## 步骤4

![](https://gitee.com/xinyongp/image/raw/master/20201207100617.png)

健康值为单浮点数，弹药值为双精度浮点数，精确搜索可以直接找到

![](https://gitee.com/xinyongp/image/raw/master/20201207100721.png)



## 步骤5

![](https://gitee.com/xinyongp/image/raw/master/20201207100824.png)



查找数值，然后找出是什么修改了这个地址

![](https://gitee.com/xinyongp/image/raw/master/20201207101450.png)

然后将其替换为nop

![](https://gitee.com/xinyongp/image/raw/master/20201207101534.png)

此时会发现无法修改数值了

![](https://gitee.com/xinyongp/image/raw/master/20201207101615.png)



## 步骤6

![](https://gitee.com/xinyongp/image/raw/master/20201207101652.png)



1. 找到精确的数值

![](https://gitee.com/xinyongp/image/raw/master/20201207102321.png)

2. 找出是什么改写了这个地址



![](https://gitee.com/xinyongp/image/raw/master/20201207102404.png)

可以看到，此时的 eax 为 0x29b，十进制667

而edx则是一个指针，接下来我们要查找这个指针数值存放的位置

3. 查找指针数值存放位置

![](https://gitee.com/xinyongp/image/raw/master/20201207102623.png)

可以看到存放 `001EC8E8` 数值的地址为 `006426B0`，手动添加这个地址

4. 手动添加指针

![](https://gitee.com/xinyongp/image/raw/master/20201207102736.png)

可以看到手动添加的指针

![](https://gitee.com/xinyongp/image/raw/master/20201207102751.png)

修改值为5000，并锁定

此时点击修改指针，成功

![](https://gitee.com/xinyongp/image/raw/master/20201207102843.png)



## 步骤7

![](https://gitee.com/xinyongp/image/raw/master/20201207102917.png)



查找到数值之后，找出是什么改写了这个地址：

![](https://gitee.com/xinyongp/image/raw/master/20201207103827.png)



点击显示反汇编程序

![](https://gitee.com/xinyongp/image/raw/master/20201207103858.png)



然后点击代码注入

![](https://gitee.com/xinyongp/image/raw/master/20201207103926.png)

修改后的代码如下

```
alloc(newmem,2048)
label(returnhere)
label(originalcode)
label(exit)

newmem: //this is allocated memory, you have read,write,execute access
//place your code here
ADD [ebx+000004A4], 3
originalcode:
sub dword ptr [ebx+000004A4],01

exit:
jmp returnhere

"Tutorial-i386.exe"+275E3:
jmp newmem
nop 2
returnhere:

```

![](https://gitee.com/xinyongp/image/raw/master/20201207104040.png)

成功

![](https://gitee.com/xinyongp/image/raw/master/20201207104105.png)



## 步骤8

![](https://gitee.com/xinyongp/image/raw/master/20201207104130.png)

