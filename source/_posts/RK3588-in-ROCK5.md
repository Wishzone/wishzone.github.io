---
title: RK3588_in_ROCK5
date: 2022-12-10 20:59:07
tags: ROCK5B
categories: LINUX
---
# 性能参考

[https://browser.geekbench.com/v5/cpu/18817954](https://browser.geekbench.com/v5/cpu/18817954)
[![img](https://shjdgwj.github.io/14733f4e12d9/image-20221123144235521.png "image-20221123144235521")](https://shjdgwj.github.io/14733f4e12d9/image-20221123144235521.png)

# 命令

<!--more-->

## 抽奖

```shell
   for i in `seq 0 7`; do echo ========cpu$i;cat /sys/bus/cpu/devices/cpu${i}/cpufreq/scaling_max_freq; done
   sudo sed -i "s|focal|jammy|g" /etc/apt/sources.list && sudo apt update && sudo apt dist-upgrade && sudo reboot
   cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq
   lscpu | grep MHz #pytm-volt-sel 越大越好
   dmesg | grep -i pvtm
   dmesg | grep -E 'pvtm|dmc' | grep sel
```

   [![img](https://shjdgwj.github.io/14733f4e12d9/image-20221026213053527.png "image-20221026213053527")](https://shjdgwj.github.io/14733f4e12d9/image-20221026213053527.png)

## 中文

```shell
   sudo apt install locales
   sudo apt install  fonts-noto-cjk fonts-noto-cjk-extra
   sudo dpkg-reconfigure locales 
```

   选 en_US UTF-8 和 zh_CN UTF, 设置默认locales 为 en_us

## 工具箱

```shell
   wget -O rockpi5.sh https://uwp.cc/s/board/rockpi5.sh 
   sudo chmod +x rockpi5.sh
   sudo ./rockpi5.sh
```

## PD协商电压 温度

```shell
   sudo apt install lm-sensors
   sensors
```

## 首次安装

```shell
   sudo apt install build-essential #编译
   sudo apt install ntp#网络校时
```

# 方法

## 如何在5B上使用桌面GPU加速

1. 安装kde
2. 安装libqt5gui5-gles, libqt5quick5-gles, libqt5quickparticles5-gles
3. 在/etc/profile里增加export KWIN_COMPOSE=O2ES
4. 重启，选择kde登录
   然后你就能享受到虽然还是卡但是是有GPU加速的桌面了

## Rock 5B 本地编译要点说明

   最重要的差别是 ： （阿超哥的方法 ）检测编译环境，如果是在 arm64上编译的话，就去掉ARCH 和 CROSS_COMPILE 交叉编译变量 build/board_configs.sh 最后面加上这几行：

```shell

   #build on native arm64
   if [ "X$(uname -m)" == "Xaarch64" -a "X${ARCH}" == "Xarm64" ]; then
       unset ARCH
       unset CROSS_COMPILE
   fi
```

* 主要参考是官方wiki：[https://wiki.radxa.com/Rock5/guide/build-debian-from-debos-radxa](https://wiki.radxa.com/Rock5/guide/build-debian-from-debos-radxa) （这里原文是跨平台交叉编译）
* 在工具链这里，不需要 安装跨平台工具链 gcc-arm-10.3***** ，只要本地安装 gcc 及配套工具（ sudo apt-get install gcc xxx)
* 其他按官方顺序执行命令
  注意：u-boot 和kernel有配置好的config，只要按官方分别执行对应的 ./build/mk-uboot.sh rk3588-rock-5b 或 ./build/mk-kernel.sh rk3588-rock-5b

## exFAT

   安装exfat fuse

## kubesphere 3.3.1

   步骤1：更换内核
   因为 Radxa 官方内核现在不支持，暂时需要更换内核：
   [https://github.com/ihexon/rock5b_kernel/releases](https://github.com/ihexon/rock5b_kernel/releases)
   步骤2：根据 KubeSphere 官方文档安装
   [https://kubesphere.com.cn/docs/v3.3/quick-start/all-in-one-on-linux/](https://kubesphere.com.cn/docs/v3.3/quick-start/all-in-one-on-linux/)
   步骤3：修补 KubeSphere 开源版暂不支持的 arm64 镜像

```shell
   # defaultbackend
   sudo docker pull playgali/defaultbackend
   sudo docker tag playgali/defaultbackend mirrorgooglecontainers/defaultbackend-amd64:1.4
```

## 驱动

   [https://github.com/happyme531/Adafruit_Blinka/commits/rk3588-rock-5](https://github.com/happyme531/Adafruit_Blinka/commits/rk3588-rock-5)
   rock5 gpio i2c spi pwm 串口 adc 全部适配完成

## wifi

   使用 `nmtui`注意Security选择 WPA & WPA2 Personal

## 中文设置

```shell
   sudo apt-get install locales
   # 配置locales
   sudo dpkg-reconfigure locales
```

   把 `en_US.UTF-8`、`zh_CN`的4个全部选上，最后一步设置默认locales 那里选 `zh_CN.UTF-8`

   安装中文字体

```shell
   apt install fonts-noto-cjk
```

## KDE桌面

   [Using different desktop environments on Armbian](https://forum.armbian.com/topic/10526-using-different-desktop-environments-on-armbian/)

```shell
   #安装KDE桌面
   sudo add-apt-repository ppa:kubuntu-ppa/backports
   sudo apt-get update
   sudo apt-get install -y kubuntu-desktop
   #修改默认桌面
   cd /usr/share/xsessions
   #看到有ubuntu.desktop  ubuntu-xorg.desktop
   #后面还不会改
```

## 安卓root

```shell
   #备份boot 
   dd if=/dev/block/nvme0n1p5 of=/storage/emulated/0/mod.img
   #刷入boot 
   dd if=/storage/emulated/0/mod.img of=dev/block/nvme0n1p5
    sync
    reboot
```

# 系统

1. [rock5b uos](http://www.zphj1987.com/rock5b-ke-yi-yun-xing-deuos-ca.html)
2. rock5b专用内核：完美收官，对应源码已上传：
   [https://github.com/unifreq/linux-rock5b](https://github.com/unifreq/linux-rock5b)
   允许全频率（电压超至1.1v, 默认未开）
   开启方法：
   nano /boot/armbianEnv.txt
   把overlays=uart7-m2改为overlays=uart7-m2 [空格] full-cpufreq
3. Rock5b Openwrt固件：采用5.10.149内核，并支持nvme（已测试）或usb启动（未测试）。
   如果需要nvme或usb启动，必须要先刷入spi 的bootloader, 用前两天发的 bootloader包：
   rock5b-bootloader_20221017.tar.gz
   [https://t.me/openwrt_flippy/3464](https://t.me/openwrt_flippy/3464)
   用法：
   1. 先刷入TF卡启动成功后，把img镜像上传到 /mnt/mmcblk0p4
   2. 进入TF卡里的openwrt,运行命令：
      cd /mnt/mmcblk0p4
      dd if=openwrt-xxxxxx.img of=/dev/nvme0n1 bs=1M conv=fsync
   3. 然后关机，把TF卡拔掉再开机即可。
   4. 用同一个镜像写的TF卡固件和nvme固件由于UUID相同，如果再重新插上TF卡启动会有问题，TF和nvme采用不同版本的固件就没问题。
4. 官方镜像用户名密码 rock @rock
5. > 目前我看到的并且我自己试过的各种镜像，我总结一下，咱们群里用的各种rock5b镜像主要是有四个来源：
   >
   > 1、radxa官方发出的debian、ubuntu和安卓，已经有半个月没更新了，debian和ubuntu可以直接装obs，但没有mali驱动之类的，不能用gpu
   >
   > 2、armbian官方社区发出的armbian，尤其是sid的那个，但armbian官方社区也已经有半个多月没更新了，我试过记得也可以直接装obs，但也没有mali驱动之类的，不能用gpu
   >
   > 3、咱们群里有lost in utopia这位大佬，整合出来mali驱动，可以用gpu，并且这位大佬在3天前又发出自带mali驱动可以开箱即用的armbian镜像，桌面是gnome，doom3可以全特效平均跑40帧，但不能直接装obs
   >
   > [https://github.com/amazingfate/armbian-rock5b-images/releases](https://github.com/amazingfate/armbian-rock5b-images/releases)
   >
   > 4、咱们群里还有StatusHeadcrabed这位大佬， 也发出自带mali驱动可以开箱即用的armbian镜像，桌面是gnome和kde，doom3可以全特效平均跑40帧，但不能直接装obs
   >

# 驱动

1. RTL8852be
   下载[rtl8852bu_fw.bin](https://github.com/armbian/firmware/blob/master/rtl8852bu_fw)放进/lib/firmware/realtek
2. 显卡GPU
   * [Fun with CSF firmware](https://icecream95.gitlab.io/fun-with-csf-firmware.html)
   * [mesa](https://gitlab.com/panfork/mesa/)

     ```shell
     sudo add-apt-repository ppa:liujianfeng1994/panfork-mesa
     sudo apt update
     sudo apt install libgl1-mesa-dri
     sudo apt-cache policy  libgl1-mesa-dri
     sudo apt install mesa*
     ```
   * [panfwost](https://gitlab.com/panfork/panfwost)

## 参考资料链接
