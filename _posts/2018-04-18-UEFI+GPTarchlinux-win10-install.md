---
layout: post
title:  "UEFI+GPT分区的archlinux和win10双系统安装"
date: 2018-04-18 16:25:06 -0700
---

UEFI是什么？
====
　　UEFI，和EFI是一个意思。EFI和BIOS的作用类似。BIOS自从PC问世以来就存在于系统中而被大家熟知，但是实在有点老迈年高了，有很多的限制。于是IT巨头又发明了EFI来取代了原来的BIOS，并且能干更多的事情。对于BIOS的主板来讲，BIOS只能完成非常基本的硬件检测和初始化，以后的工作就都由bootleader（如GRUB,LILO等）来交接了；对于支持UEFI的主板，主板的ROM中存放EFI shell程序，会能够识别存储介质上的分区信息和文件系统，比如fat32，并从指定的EFI/boot/目录下查找.efi文件，并执行。

GPT是什么？
====
　　GUID磁盘分区表（GUID Partition Table，缩写：GPT）其含义为"全局唯一标识磁盘分区表"，相对于传统的分区表叫MBR(使用FDISK分区软件)，其最大特点是支持2T以上的分区。如今硬盘愈来愈大，如果你想划一个超过2T的分区，以前的MBR就表示无能为力了，只能用GPT分区形式。
　　为什么会有这样的场景？  
　　最近本人想搭建一个私有云服务器，购买了J3455低功耗芯片的主板，购买时说明主板只能支持win10。另外私有云准备安装archlinux系统。既然搭建私有云当然可能会有大于2T的需求，所以必须采用GPT分区。同时鱼与熊掌想兼得，也要安装win10时来感受一下J3455cpu的4k解码，所以只能安装双系统了。  
　　具体GPT的好处和各个windows版本的兼容情况可以参考baidu(https://baike.baidu.com/item/GPT%E7%A3%81%E7%9B%98/12761414?fr=aladdin)。 

安装过程
====
1. 准备分区  
　　GPT分区不能用常用的fdisk分区工具，最简单的使用parted分区工具。这里最主要的是对磁盘新增一个ESP分区。EFI系统分区，即 EFI system partition，简写为 ESP，这是EFI 规范规定的，并且ESP分区必须使用FAT格式，是强制性的，一般分200M-500M大小就可以了。
　　在bios设置使用uefi启动，并且设置u盘启动优先，用aurchlinux的安装盘启动电脑。
　　#parted mklabel gpt
　　#parted mkpart primary fat32 0 500M (创建esp分区)
　　#mkdosfs /dev/xxx (格式化esp分区)
　　其他分区可以这里事先分好，或者待下面安装操作系统时分也可以。
1. 安装win10  
　　找一个大于8G的U盘，到网上找一下u盘安装工具，做一个系统启动U盘，并下载win10软件到U盘。网上有不少教程，这里不细说了，但要找支持UEFI启动的U盘制作工具。
1. 安装archlinux  
　　在安装完win10之后，可以在bios可以设定win10启动或者uefi启动，这里需要选择uefi启动。再次使用archlinux的安装u盘启动，按常规安装archlinux，参考https://wiki.archlinux.org/index.php/Installation_guide。
1. 安装启动程序  
　　安装还没退出chroot环境执行以下命令：（如果重新启动电脑了，请用arch-chroot /mnt再进入chroot环境）  
　　```#pacman -S grub efibootmgr os-prober (安装grub/efibootmgr/os-prober)```  
　　挂载 ESP 分区，挂载到 /boot/efi。将下面命令中的 esp_mount 修改为挂载点/boot/efi  
　　```#grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub```  
　　执行是需要安装efibootmgr包，--bootloader-id 是启动项中用来识别 GRUB EFI 的标记，请用容易判断的信息。安装程序会在 esp/EFI/ 的相同目录创建相同名称的目录用来存放 EFI 二进制启动加载器。
　　上述安装完成后 GRUB 的主目录将位于 /boot/grub/,会在ESP分区生成/EFI/grub/grubx64.efi，并生成/boot/grub/x86_64-efi/  
　　生成主配置文件  
　　```#grub-mkconfig -o /boot/grub/grub.cfg```  
　　grub-mkconfig会自动搜索已经安装的其它系统并添加到启动菜单(之前已经挂载的/boot/efi下会找到win10的启动项)  ，如果没法找到win10启动项，可以重启电脑之后再挂载EFI分区，重新执行该命令。每次修改 /etc/default/grub 后，都需要重新#生成主配置文件.  
　　如果顺利，再次启动电脑可以看到grub启动菜单有archlinux和windows两个启动项，整个安装过程就完成了。如果你安装debain这种linux发行版，最后系统应该自动配置启动项，不过我没试过不能妄下结论。

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)

