---
layout: post
title:  "MicroPython1.9发布"
date: 2017-05-30 16:25:06 -0700
---

　　2017年5月26日MicroPython官方网站发布了MicroPython1.9，今天（5月30日）很多硬件的固件版本又进行了一次更新，特别是官方网站第一次公布了ESP32的固件，真是普大喜奔，小伙伴快来看看1.9版本有什么主要的变化。  
1. 此版本增加了一些基本的新组件，允许更多的Python操作不使用堆的操作，减少了代码大小和堆栈使用。
2. 一个新的通用VFS子系统，允许安装任意文件系统（甚至是用Python写的）在根或任意点挂载。FatFs驱动已经替换为一个面向对象的版本（oofatfs），允许完全可定制文件系统层次结构。
3. 调度框架已经在核心实现，提供"尽快"的回调功能。这使得端口可以实现软IRQ中断处理。新micropython的schedule()函数可以完成调度功能。
4. 多线程已经通过_thread模块实现，但此功能默认情况下禁用，必须在编译时启用
micropy_thread和micropy_thread_gil选项
5. 其它包括：常量可以是一个big-number类型；增加了一个帮助工具mpy_cross_all.py；字节代码格式有变化，原来的.mpy文件需重新编译。有些C api的改变，比如mp_uint_t改成size_t等。
进一步的介绍和具体各版本的改变请参考：
http://micropython.org/resources/micropython-ChangeLog.txt

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)

