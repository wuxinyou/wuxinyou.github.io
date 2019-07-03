---
layout: post
title:  "STMCU程序ISP下载方式"
date:   2019-01-02 11:34:54
categories: 嵌入式开发
tags: 原创 STMCU开发
author: wuxy
---

* content
{:toc}

本文分为2部分，第一部分是简单的讲述利用flyMcu上位机和ISP的方式下载程序。第二部分的比较有含金量，主要讲述ISP下载的具体过程，该部分可是自己编写类flyMCU的上位机。

本文测试的目标板为stm32103RcT6。

# 利用flyMcu下载程序

## ISP与IAP

>
Iap,全名为in applacation programming,即在应用编程,与之相对应的叫做isp,in system programming,在系统编程,两者的不同是isp需要依靠烧写器在单片机复位离线的情况下编程,需要人工的干预,而iap则是用户自己的程序在运行过程中对User Flash 的部分区域进行烧写，目的是为了在产品发布后可以方便地通过预留的通信口对产品中的固件程序进行更新升级。在工程应用中经常会出现我们的产品被安装在某个特定的机械结构中,更新程序的时候拆机很不方便,使用iap技术能很好地降低工作量.

## ISP下载前准备
- 编译好的hex文件；
- 电脑以及FlyMcu上位机；
- USB-TTL

## ISP下载方式
- Boot0 接到 3.3V 上，Boot1 接到 GND，对板子重新上电，STM32 单片机重启的时候，会进入到 ISP 模式;
- USB-TTL 对接目标板的uart1;
- 打开FlyMcu上位机软件，如下图操作;
![operation](/images/2019-1-02-embededDev-STMCU-ISPDownload-1.png)
- Boot0和boot1都接到GND,目标板重新上电，或者Rst，即可正常运行程序。

## 注意事项
- ISP下载需要在单片机离线情况下才能下载，所以需要Rst操作;
- 正常运行时，需要重新设置boot0,boot1;
- IPS下载实际上使用的串口下载，测试时使用的是uart1,理论上uart1在程序中可以正常使用的，因为下载时会将单片机离线;
- 由于ISP下载是串口下载，所以对PC主机要求低，不需要安装Jtag等。


## 参考阅读
- [STM32_IAP详解(有代码,有上位机)](https://www.cnblogs.com/dengxiaojun/p/4336239.html)

# ISP下载过程

## 参考阅读
- [STM32 ISP烧录过程](https://blog.csdn.net/cao_yanjie/article/details/76269049)
