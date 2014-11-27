---
layout: post
title:  "搭建 raspberry pi (raspbian) 开发环境"
date:   2013-01-12 17:47:45
categories:
---

这几天想在 rpi 上建立 nginx + nodejs + redis-server 的服务，而我想用 最新版的 nginx 和 nodejs 想来得自己编译。

编译软件有两种方式，一种直接在 rpi 上编译，另外在自己的笔记本上编译，然后 cp 到 rpi

笔记本用的是 64 位 的 Ubuntu 12.04 所以直接在 笔记本上建立 chroot 的 环境。

# 安装 QEMU

因为使用的是不同的硬件架构，所以需要一个模拟器，我选择 QEMU :

{% highlight bash %}
    sudo apt-get install qemu-user-static
{% endhighlight %}

# 建立 chroot 环境

建立 chroot 环境（使用 using qemu-debootstrap ）并挂载 所需要的 文件系统。

{% highlight bash %}
sudo apt-get install debootstrap
sudo qemu-debootstrap --arch armhf wheezy chroot-raspbian-armhf http://mirror.nus.edu.sg/raspbian/raspbian
sudo mount -t proc proc chroot-raspbian-armhf/proc
sudo mount -t sysfs sysfs chroot-raspbian-armhf/sys
sudo mount -o bind /dev chroot-raspbian-armhf/dev
{% endhighlight %}

记住这里只建立最基本的系统，你必须自己配置所有的东西，包括建立用户，安装软件等

# chroot

现在你可已经入 chroot 环境。

{% highlight bash %}
sudo LC_ALL=C chroot chroot-raspbian-armhf
{% endhighlight %}

# 一些简单的配置

## 建立用户

{% highlight bash %}
useradd -m -s /bin/bash <username>
{% endhighlight %}
## 配置源 和 公钥

{% highlight bash %}
echo "deb http://mirror.nus.edu.sg/raspbian/raspbian wheezy main" >> /etc/apt/sources.list
wget http://archive.raspbian.org/raspbian.public.key -O - | apt-key add -
apt-get update
{% endhighlight %}

## VideCoreIV userspace libs

You may need to fetch the libraries for VideoCoreIV for hf, these live here: https://github.com/raspberrypi/firmware/tree/master/hardfp

# 重启电脑

重新 挂载 文件系统

{% highlight bash %}
sudo mount -t proc proc chroot-raspbian-armhf/proc
sudo mount -t sysfs sysfs chroot-raspbian-armhf/sys
sudo mount -o bind /dev chroot-raspbian-armhf/dev
{% endhighlight %}
