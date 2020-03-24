---
layout: post
title:  "关于STM32开发的一些问题"
date:   2019-01-18 9:22:54
categories: 嵌入式开发
tags: 原创 MCU开发
author: wuxy
---

* content
{:toc}

本文记录stm32开发过程中的一些疑问和心得。

## 关于时钟
- stm32的时钟来源都有哪些，分别如何配置，在哪些关键寄存器中配置，用哪些基本的库函数？
- 什么时候用外部高速时钟(HSE),什么时候用HSI，软件如何配置，硬件又是如何自行置/复位。
- SystemInit()函数是如何设置所有的时钟的？
- 如果使用系统内部的时钟(HSI),即外部不需要外接晶振了，软件该如何设置？既然有内部时钟，为什么还要外部晶振呢？
- PLL倍频是如何工作的？软件如何设置PLL倍频？

以上问题的参照《STM中文参考手册》时钟部分，可以基本解决。

## 关于定时器
- 定时器有哪3种，分别是哪些？
- 通用定时器(T2~T5)都有哪些功能，如何配置？
- 以T3为例，T3有4个独立的通道，为什么中断却只有一个？
- 定时器在不同功能时，中断时怎么工作的？

## printf重映射
大概有两种方法，无论是那种方法，都务必USART_Configuration()，即配置串口输出。
- https://blog.csdn.net/qq_29344757/article/details/75363639 ： STM32的printf函数重定向，如果找不到USART_SendChar（）函数，就请用USART1->DR = (u8) ch;另外如果是C函数在.CPP文件中，务必包含在extren C内。亲测有效。
- https://blog.csdn.net/liuxiuqi19860119/article/details/84678634 ： STM32-printf重映射串口，没试过，但是看过很多源码使用这种方法。
