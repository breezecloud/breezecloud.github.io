---
layout: post
title:  "给树莓派安装ArchLinux"
date: 2017-12-15 16:25:06 -0700
---
&emsp;&emsp;ArchLinux是Linux的一个发行版，以简单、轻量为其设计理念。由于其小而美的特点，ArchLinux非常合适运行在树莓派等环境上，树莓派官方网站曾经提供了ArchLinux的镜像下载，不知道为何后来又取消其链接，也许是arm版本的ArchLinux本身不是官方版本的缘故，不过在http://archlinuxarm.org/还提供对树梅派的支撑，让我们一起来看看如何安装和配置。

第一部分：安装
===
&emsp;&emsp;下载最新版的ArchLinux for Arm的系统:浏览http://archlinuxarm.org/os/ 可以找到适合的树梅派版本的安装文件。md5sum可以校验一下下载的文件是否有问题。
1. 第一步：用fdisk给SD卡进行分区:  
fdisk /dev/sdx
进入fdisk的界面后，要删除原有分区，创建新的分区
* 输入o并回车，清空原有分区。
* 输入p列出当前分区，此时应该看不到任何分区。
* 输入n，回车然后输入p选择建立主分区，然后输入1，然后直接回车选择默认的起始扇区，然后输入+100M为结束的扇区(为第一个分区指定100MB的空间)。
* 依次输入t，c，设置第一个分区的分区类型为W95 FAT32 (LBA)。
* 依次输入n，p建立第一个主分区，然后输入2为其分区号，然后两次回车接受默认起止扇区设置。
* 输入w以写入分区数据并退出fdisk。
2. 第二步: 创建并挂载文件系统  
```
mkfs.vfat /dev/sdx1
mkdir boot
mount /dev/sdx1 boot
mkfs.ext4 /dev/sdx2
mkdir root
mount /dev/sdx2 root
```
3. 第三步: 解压文件到root目录  
```
tar xzvf ArchLinuxARM-rpi-latest.tar.gz -C root
sync
mv root/boot/* boot
umount boot root
```
最后把SD卡插入树莓派，接入电源即可点亮。等等，怎么没反应。是不是用了HDMI转VGA的线？老规矩在boot分区的config.txt文件加上：
```
hdmi_force_hotplug=1
config_hdmi_boost=4
hdmi_group=2
hdmi_mode=9
hdmi_drive=2
hdmi_ignore_edid=0xa5000080
```
&emsp;&emsp;再试试启动树莓派是否正常。
注意：你的系统中，需要把上面的sdx替换成相应的编号。系统root用户的默认密码是root，root用户默认不能远程登陆，可以使用alarm用户登陆，其默认密码为alarm。

第二部分：设置网络
===
archlinux如何简单配置网络？archlinux的网络配置和其它发行版有很大不同。
修改：
```
/etc/systemd/network/eth0.network
[Match]
Name=eth0
[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
```
vim /etc/systemd/resolved.conf
```
[Resolve]
DNS=114.114.114.114

rm -f /etc/resolv.conf
ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf 
systemctl disable netctl.service
systemctl enable systemd-networkd.service
systemctl enable systemd-resolved.service
reboot
```
&emsp;&emsp;然后重启一下试试，可以用ip addr 或者ifconfig查看网卡状态。
另外无线配置可以使用wifi-menu启动菜单配置。

第三部分：备份和恢复系统
===
&emsp;&emsp;从安装系统的方式受到启发，可以用tar来备份整个系统，这种方式不仅仅局限于ArchLinux，其它版本也应该适用。这样万一需要恢复系统，不需要从安装盘开始"重走长征路"，也不需要用dd命令将整个sd卡备份下来形成一个巨大的文件费事费力。
* 备份：  
　　找一个容量比较大的u盘，删除原来的分区并格式化成ext4（ArchLinux默认不支持nt格式）。
```
mount /dev/sda1 /mnt
cd /mnt
tar -czvf archlinux_pi2_20171213.tar.gz --exclude=/media/* --exclude=/sys/* --exclude=/proc/* --exclude=/mnt/* --exclude=/tmp/* /
```
* 恢复：  
　　分区和格式化操作和前面安装系统时是一样的。将sd卡分成两个分区，一个用于启动分区（100M），剩余空间全部给根分区。如果已经分过区，这步不需要操作，直接格式化。
```
cd /mnt
mkdir boot
mount /dev/sda1 boot
mkdir root
mount /dev/sda2 root
tar -xzvf archlinux_pi2_20171213.tar.gz -C root
mv root/boot/* boot
unmount root boot
```

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)
