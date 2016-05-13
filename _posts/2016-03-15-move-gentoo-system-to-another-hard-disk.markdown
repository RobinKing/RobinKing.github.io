---
layout: post
title:  "移动Gentoo到新硬盘"
date:   2016-03-15 23:00:00
categories: [Computer Science]
tags: [Linux ,Gentoo]
---
### 序：背景
******
最近硬盘空间又不够用了，看着京东开学季有优惠，剁手族当然又忍不住了。俺的本本用了也快七八年了，再往上面添一块SSD还是有点心疼的，所以还是入手一块1TB的机械硬盘。

考虑到光驱现在基本也都是用不到了，顺带买了个光驱位的硬盘托架，哈哈，两块硬盘应该足够了。

由于在学校和工作中都离不开Windows，有时间也会写一些win的程序，所以俺一直是某Linux发行版
\+ Windows 双系统。现在用的就是Gentoo \+ Win
10，既然现在有两块硬盘了，当然可以让二者各自使用独立的硬盘了，所以俺计划将Gentoo系统搬家到新硬盘，原来的老硬盘就去光驱位咯。哇咔咔~~~~

那么如何将现有硬盘上的Gentoo系统“移动”到新硬盘呢？

### 〇：准备工作
---

首先将新买的硬盘装在光驱位开机（I
guess如果使用移动硬盘盒滴话也是可以滴）。然后就可以进行磁盘准备工作啦～～～～

既然这次Linux使用单独的硬盘，所以俺就打算使用GPT分区表了。

{% highlight bash %}
# fdisk 命令
$ fdisk -t gpt /dev/sdb
# parted 命令
$ parted -a optimal /dev/sdb
...
(parted) mklabel gpt
{% endhighlight%}

使用parted或者fdisk等工具，将新硬盘分区。
俺最终创建的硬盘分区如下所示。

|分区       |文件系统   |描述       |
|------------------|-----------------|-----------------|
|/dev/sdb1 |(bootloader)|BIOS启动|
|/dev/sdb2 |ext2        |boot   |
|/dev/sdb3 |(swap)      |交换分区|
|/dev/sdb4 |ext4        |根目录 |
|/dev/sdb5 |ext4        |usr目录 |
|/dev/sdb6 |ext4        |home   |
|/dev/sdb7 |ntfs        |备用   |

### 一：挂载分区 并 复制文件
---
挂载上一步创建的分区。
{% highlight bash %}
# !/bin/bash
mount /dev/sdb4 /mnt/gentoo
mount /dev/sdb2 /mnt/gentoo/boot
mount /dev/sdb5 /mnt/gentoo/usr
mount /dev/sdb6 /mnt/gentoo/home
...
{% endhighlight %}
复制除/dev /run /proc /sys 之外的文件夹到刚才挂载的分区。
PS.此处应该可以使用dd - -
{% highlight bash %}
# !/bin/bash
mount /boot
cp -iva /boot/* /mnt/gentoo/boot
...
{% endhighlight %}
到这一步，系统的搬家工作就结束了。在新硬盘上安装grub，更换硬盘后即可使用咯～～～

### 二：挂载 并 chroot
---
挂载其他必须的文件。
{% highlight bash %}
# !/bin/bash
mount -t proc proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
{% endhighlight %}
chroot到新硬盘。
{% highlight bash %}
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot)"
{% endhighlight %}
### 三：安装grub
---
安装grub到新硬盘。
{% highlight bash %}
(chroot) grub2-install /dev/sdb
(chroot) grub2-mkconfig -o /boot/grub/grub.cfg
{% endhighlight %}

### 四：编辑fstab 并 更换硬盘
---
编辑fstab文件。
{% highlight bash %}
#/etc/fstab
/dev/sda2		/boot		ext2		noauto,noatime	1 2
/dev/sda4		/		ext4		noatime		0 1
/dev/sda3		none		swap		sw		0 0
/dev/sda5		/usr		ext4		noatime		0 0
/dev/sda6		/home		ext4		noatime		0 0
{% endhighlight %}

交换硬盘，将新硬盘放置在硬盘位，将windows所在的硬盘放置在光驱位。

### 结束
---
重启系统，设置bios，使用新硬盘启动。进入系统后，使用os-probe识别硬盘sdb上的windows系统。
{% highlight bash %}
sudo os-probe
{% endhighlight %}

尽情地在新硬盘里畅游吧～～～～～～～～～～～～
