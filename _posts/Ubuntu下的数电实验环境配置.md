---
title: Ubuntu下的数电实验环境配置
tags: [DLCO]
date: 2022-03-08 13:06:00
---
\newcommand\norm[1]{\left\lVert#1\right\rVert}
\newcommand\abs[1]{\left\lvert#1\right\rvert}
针对南京大学 数字逻辑与计算机组成实验 课程的环境配置，本机是Ubuntu 21.10


# 1


访问
[这个网页](https://www.intel.com/content/www/us/en/software-kit/660904/intel-quartus-prime-lite-edition-design-software-version-20-1-1-for-linux.html?)，选择Individual
Files，只需要下载


1. QuartusLiteSetup-20.1.1.720-linux.run (1.9GB)

2. cyclonev-20.1.1.720.qdz (1.3GB)




# 2


进入下载目录，执行以下操作


```bash

chmod +x QuartusLiteSetup-20.1.1.720-linux.run

./QuartusLiteSetup-20.1.1.720-linux.run

```


途中选择带有 Free License 字样的 Modelsim-Questa




# 3


再参照[这个回答](https://stackoverflow.com/questions/31908525/modelsim-altera-error)，命令行执行以下操作


```bash

sudo apt-get install libxft2 libxft2:i386 lib32ncurses6

```


第二个是必须的，这样就解决了 RTL Simulation 时，弹窗报错需要 LD_LICENSE_FILE 环境变量的问题。




# 4


参照[Intel官方文档](https://www.intel.com/content/www/us/en/docs/programmable/683076/current/installing-on-red-hat-enterprise-5-and-6.html)，以su权限修改`/etc/udev/rules.d/51-usbblaster.rules`文件，添加：


```bash

# Blaster I

BUS=="usb", SYSFS{idVendor}=="09fb", SYSFS{idProduct}=="6001", MODE="0666"

BUS=="usb", SYSFS{idVendor}=="09fb", SYSFS{idProduct}=="6002", MODE="0666"

BUS=="usb", SYSFS{idVendor}=="09fb", SYSFS{idProduct}=="6003", MODE="0666"

# Blaster II

BUS=="usb", SYSFS{idVendor}=="09fb", SYSFS{idProduct}=="6010", MODE="0666"

BUS=="usb", SYSFS{idVendor}=="09fb", SYSFS{idProduct}=="6810", MODE="0666"

```


具体关于udev是干什么的，可以看archwiki




# 5


在`/usr/share/applications/`目录下以su权限新建`quartus.desktop`，输入以下内容


```bash

[Desktop Entry]

Type=Application

Version=0.9.4

Name=Quartus (Quartus Prime 20.1) Lite Edition

Comment=Quartus (Quartus Prime 20.1)

Icon=/home/jjppp/intelFPGA_lite/20.1/quartus/adm/quartusii.png

Exec=/home/jjppp/intelFPGA_lite/20.1/quartus/bin/quartus --64bit

Terminal=false

Path=/home/jjppp/intelFPGA_lite/20.1

```


这样就可以找到quartus作为程序的图标了




# 6


可以安装wine，那么就可以利用System Builder来生成已经分配好引脚的工程文件了。




# 总结


中途遇见了很多奇怪的问题，最奇怪的是LD_LICENSE_FILE的问题，明明是免费版本却出现了需要一个不存在的license.dat文件的情况.....最后是通过"Questa-Modelsim
LD_LICENSE_FILE"搜到的解决方案，不然就得去翻log了。
