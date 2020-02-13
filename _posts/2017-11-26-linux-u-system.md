---
layout: post
title:  "制作基于u盘的linux系统"
date: 2017-10-24 20:25:06 -0700
---


1. 为何要制作基于u盘的linux系统  
&emsp;&emsp;虽然现在手机平板电脑已经取代了pc的一部分功能。但有时候全功能的pc还是需要的。特别是你需要在不同的地方使用pc又不想老背着台笔记本。其次碰到紧急情况可以应急一下充当急救盘。做一个随身的linux系统。 只要有一台电脑。。插入u盘就能any time any where的使用linux。 也是不是符合自由软件的精神？这也是我从ideal到实现的一次尝试。  
为了做一个基于u盘的linux系统。 我在网上搜了一下。 结果大部分是使用某个发行版的live启动盘做启动u盘（直接将镜像文件写入u盘）。这种方式无法当普通系统使用。 后来只找到一篇博文讲实现的一些思路。 所以决定亲手实践一下。以下操作经过本人验证。 有些坑已经踩过希望能帮助到有相同想法的人。
2. 系统选择  
&emsp;&emsp;有一些专为U盘构建的Linux系统。 如试用过puppy。 这类系统确实很方便。 但作为长期使用的个人系统是不太明智的。 为了精简体积。 它们在系统架构上与常规Linux有很大不同。 遇到问题不容易获得别人的帮助（因为用的人少）。 软件仓库的数量也无法同主流发行版相比。别人apt-get就能安装的软件。 你可能得花几个小时 configure && make。 就别提U盘那缓慢的IO导致的糟糕的编译速度了。所以。 我倾向于选择一个成熟的。 完整的。 主流的发行版。我的选择是Debian。Debian足够主流。 是很多发行版的鼻祖。 有很好的包管理工具方便定制。 同时又稳定纯净。如果正好和你日常试用的发行版一样。 你就不需要因为换了发行版而需要重新学习。
3. 写安装镜像到u盘  
&emsp;&emsp;准备2个u盘（一个做安装盘。 一个做目标盘）和一台运行windows的电脑（能u盘启动）。下载写入工具Win32DiskImager和debain安装盘。用Win32DiskImager将下载的安装盘镜像文件写入u盘。下载的文件后缀是iso。 不是Win32DiskImager的默认文件类型。 但是没有关系。 在选择文件时下拉框选择全部类型就可以了。另外我一开始选择的安装文件是debian-9。2。0-amd64-DVD-1。iso。 发现在安装debain到安装软件的步骤时候出错。后来又下载了安装文件debian-9。2。1-amd64-xfce-CD-1。iso。 则顺利安装完成。 所有各位一定要选择文件debian-9。2。1-amd64-xfce-CD-1。iso安装。

2. 安装debain  
&emsp;&emsp;否则电脑硬盘数据会丢失）。完成安装之后试着用新的u盘启动电脑。 矣。 怎么没法启动。也不知道什么原因。 只能重新安装grub试试。所以用安装u盘重新启动电脑。 进入resuce模式。 注意此时root目录选择目标u盘的系统目录进入shell。 执行如下命令：  
grub-install --root-directory=/ /dev/sdx ;/dev/sdx目标u盘
  退出安装流程。 重新启动电脑。 grub菜单出现了。 不由松了一口气。
4. 配置  
&emsp;&emsp;如果成功启动电脑。 那么接下来你可以如下优化：
    * 安装无线网卡
    我测试的IBMx220i笔记本电脑debain无法识别无线网卡。先查看芯片型号 lspic |grep Network（可能没有lspic命令。 需要先安装）。 到https://wireless。wiki。kernel。org/en/users/Drivers/iwlwifi下载相应驱动解压其中文件。 将iwlwifi-*。ucode文件复制到 /lib/firmware后重启就可以发现可爱的无线网络图标在状态栏出现了。
    * 安装自己常用的软件
    安装自己常用的软件。 和正常安装没什么不同。 一般用apt-get install命令安装。
    * 设置ram缓存。 加快系统速度通过修改/etc/fstab来实现这一目的：
    tmpfs /tmp tmpfs size=1024m 0 0

&emsp;&emsp;现在你可以拿着u盘到任意电脑上启动了。 enjoy it！

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin。jpg)

