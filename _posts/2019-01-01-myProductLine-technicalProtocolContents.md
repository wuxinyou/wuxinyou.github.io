---
layout: post
title:  "技术方案目录"
date:   2019-01-01 19:04:54
categories: 我的产品线
tags: 原创 我的产品线
author: wuxy
---

* content
{:toc}

## 这是什么？

技术方案目录是我的产品线的总目录，该目录详细汇总了在我的产品线中会用到的技术方案。同时在该目录的细目中包好了技术方案实现的地址链接。该目录有主要有2个作用，一是作为编写、整理技术方案的总提纲，后面的技术方案总结整理工作都以此目录为准；二是作为后期产品研制时的查找实用工具。因此该目录是一个系统的，全面的，可靠的，方便查找的技术方案目录。

凡是录入该目录的，都是切实可行的技术方案，而不是未经验证的理论，也不是简单的知识点，单纯的知识点不会录入该目录，而会在其他文章中收录。界定知识点和技术方案的简单办法就是该方案是不是可以开发为一个系统或产品。所以该目录的细目并不会很多。

随着时间的推移，需求的变更，该目录会越来越丰富，越系统，越全面。

所有的技术方案都有2个状态变量：研制状态和存档状态。研制状态的值分别是：在研中，原理样机，产品化；存档状态的值分别是：收集中，已存档。

## 关于存档

和目录保持一致的，在[技术方案归档]中都有对应的技术方案资料，所以[技术方案归档]作为所有目录的实际存储地址。关于[技术方案归档]的建立，暂时没有一个完善的解决办法，因为如果把所有的技术文档都放在[技术方案归档]下，会造成一定程度的存储冗余，尤其是大文件，因为一项技术通常会包含很多资料。一般而言，我会把所有尚未完全整理的资料存放在百度网盘和移动硬盘中做双重备份。也就说着两个地方是原始资料库，但是没有整理的很好，所以不能算是归档。

另一种方法就是在[技术方案归档]中仅仅存放实际技术方案的地址链接，但是这样就失去的建立[技术方案存档]的意义。

于是这里制定一下[技术方案存档]的规范：
- 技术方案存档必须包含关于该技术的文档说明，而不仅仅是一堆原始开发资料；
- 存档必须条例清晰，必要时需要添加ReadMe文件，用以说明存档资料的使用；
- 必须包含原始开发资料，如源代码，电路图等；
- 如果原始资料超过100M，同时在其他地方有双重备份，可以不用在冗余备份，只需要说明实际存放地址即可，但是必要的技术方案文档还是要的。也就说，只要小于100M，则和技术方案相关的资料都应该按要求存档。
- 一旦存档，理论上不会修改，因为方案必须是切实可行的，当然如果实在需要完善和改进可以修改，但是必须添加修改日志和文档说明。

同样的，[技术方案存档]也会在网盘和移动硬盘中存档，其实也可以存放在github上，但是由于github的服务器在美国，而且它也是完全开放的，所以还是放在自己手里安全点。



以下是技术方案目录内容。

## 嵌入式ARM开发

## 单片机开发
在没有特别说明的情况下，这里的单片机都是指ST的MCU。

- STM32 MCU无线下载程序技术方案-ISP无线下载。
状态：在研中，收集中。
- 离线
- 超低功耗MCU+SPI显示。
- 超低功耗MCU+超低功耗无线通信（近距离）。
- Jlink_ARM_STM32仿真下载器。
状态：产品化，已存档。


## 运动控制
- 6轴机械臂运动学控制算法。
- 直流电机驱动器。

## 视觉方案


## 手机APP

## PC上位机

## 核心算法模块