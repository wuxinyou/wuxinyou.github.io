---
layout: post
title:  "extern 'C' 作用详解"
date:   2019-02-19 10:54:54
categories: 嵌入式开发
tags: 原创 MCU开发
author: wuxy
---

* content
{:toc}

## 问题背景

我们在使用keil编程时，偶尔会见到extern "C"，那么它有什么作用呢？什么时候需要添加extern "C",什么时候又不需要添加呢？

## extern "C"概述

extern "C"的主要作用就是为了能够正确实现C++代码调用其他C语言代码。加上extern "C"后，会指示编译器这部分代码按C语言的进行编译，而不是C++的。由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型也加到编译后的代码中，而不仅仅是函数名；而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般之包括函数名。

这个功能十分有用处，因为在C++出现以前，很多代码都是C语言写的，而且很底层的库也是C语言写的，为了更好的支持原来的C代码和已经写好的C语言库，需要在C++中尽可能的支持C，而extern "C"就是其中的一个策略。

## 使用场合

这个功能主要用在下面的情况：

- C++代码调用C语言代码,即在.cpp文件中直接使用C代码时，必须使用extern C.
- 在C++的头文件中使用,在C++头文件中包含了C头文件时，通常需要使用extern C.
- 在C头文件中使用，这种用法就是的该C代码模块可以直接兼容C++，当.cpp头文件引用该模块时，就不再需要添加extern C了。STM固件库中所有的C代码头文件都使用了extern C.
- 在多个人协同开发时，可能有的人比较擅长C语言，而有的人擅长C++，这样的情况下也会有用到

总之，只有在C++和C混用的时候，才会用到extern C.

为了使代码更具有可移植性，建议所有的C代码头文件都添加extren C.也就是和STM固件库的用法类似。而不是要等到引用该C代码模块是才添加。

## 常见用法举例

```
/* Define to prevent recursive inclusion -------------------------------------*/

#ifndef __STM32F4_PWMOUT_H
#define __STM32F4_PWMOUT_H

/* Includes ------------------------------------------------------------------*/

#ifdef __cplusplus
 extern "C" {
#endif
#include <stm32f4xx.h>
#include "stm32f4xx_tim.h"
#include "stm32f4xx_gpio.h"
#include "stm32f4xx_rcc.h"
#include "STM32F4_GpioInit.h"

//others C code

#ifdef __cplusplus
 }
#endif
#endif
/******************* (C) COPYRIGHT 2009 STMicroelectronics *****END OF FILE****/
```

## 常见说明

- 有的时候C代码头文件不添加extern C时，编译会报错，报错类型如下：..\output\ApolloCtrlBox.axf: Error: L6218E: Undefined symbol getADValue(unsigned char) (referred from td310app.o).而有的时候，不添加却能编译通过。

原因是引用以上C代码模块时，在C++头文件中添加了extern C.




## 参考资料
- [extern “C”的作用详解](https://www.cnblogs.com/carsonzhu/p/5272271.html)
