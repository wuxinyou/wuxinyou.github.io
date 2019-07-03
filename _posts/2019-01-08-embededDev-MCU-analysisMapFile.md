---
layout: post
title:  "解读STMcu .map文件"
date:   2019-01-08 10:54:54
categories: 嵌入式开发
tags: 编整 MCU开发
author: wuxy
---

* content
{:toc}

## 几个问题

- .map文件是什么？

.map文件是由LINK工具生成的一种文本文件，其中包含有被连接的程序的某些信息，例如程序中的组信息和公共符号信息等。map文件不是MUC特有的，也不是keil特有的，但是本文主要以keil 5编译stm32为例进行讨论。

- 为什么要了解.map文件？

简单的说几个原因：

1.根据map文件可以分析得出程序的编译和链接情况，也能分析出代码，数据，全局变量等内存情况；

2.可以完美解释"Program Size: Code=11664 RO-data=3408 RW-data=304 ZI-data=7008 "代表的意义；

3.可以从最底层分析程序的内存使用情况，比如溢出等。

总之，深入了解map文件，可以让程序员回归底层，知其然，知其所以然，而不是遇到问题总停留在表面，总是按自己猜测去理解问题。

下面提出一些简单的问题，希望读者可以带着这些问题去解读map文件。

1.全局变量什么时候保存在RAM中，什么时候保存在Flash中？

2.对于一个具体的语句，如何区分他们是code，Ro-data,Rw-data,Zi-data?

建议扩展阅读本站文章：《STMCU中RAM和Flash的研究》

## 解读stm32 .map文件
本部分转载自：[Keil综合（03）_map文件全解析](https://blog.csdn.net/ybhuangfugui/article/details/75948282#rd)

内存溢出的问题，如何分析这类问题的呢？遇到HardFault_Handler 有对map分析过吗？首先讲述一下关于map在MDK-ARM中的配置。其实，在MDK-ARM中，我们可以根据自己的情况（不同配置），在map文件中输出对应（我们需要）的内容。默认情况下，输出所有信息。

Project -> Options for Target -> Listing：会看到如下配置界面：
![配置界面](/images/2019-01-08-embededDev-MCU-analysisMapFile-1.png)

map文件里面内容大致分为五大类（按照map文件分类的顺序）：

1.Section CrossReferences：模块、段(入口)交叉引用；

2.Removing Unused inputsections from the image：移除未使用的模块；

3.Image SymbolTable：映射符号表；

4.Memory Map of theimage：内存（映射）分布；

5.Image componentsizes：存储组成大小。


下面章节就针对MDK-ARM详细讲述一下map文件里面的几大内容。

### SectionCross References：模块、段(入口)交叉引用
![2](/images/2019-01-08-embededDev-MCU-analysisMapFile-2.png)

配置中需勾选上：CrossReference



Section CrossReferences：模块、段(入口)交叉引用，指的是各个源文件生成的模块、段（定义的入口）之间相互引用的关系。


比如：

main.o(i.System_Initializes) refers tobsp.o(i.BSP_Initializes) for BSP_Initializes

意思是：

main模块(main.o)中的System_Initializes函数(i.System_Initializes)，引用（或者说调用）了bsp模块(bsp.o)中的BSP_Initializes函数。

提示：

main.o是main.c源文件生成的目标文件模块；

I.System_Initializes是System_Initializes函数的入口。

### RemovingUnused input sections from the image：移除未使用的模块

![3](/images/2019-01-08-embededDev-MCU-analysisMapFile-3.png)

配置中需勾选上：Unuaed SectionsInfo



这一选项很好理解，就是我们工程代码中，没有被调用的模块。

最后还有一个统计信息：

52 unused section(s) (total2356 bytes) removed from the image.

1.总共有52段没有被调用；

2.没有被调用的大小为2356 字节；

### ImageSymbol Table：映射符号表(重要)
![4](/images/2019-01-08-embededDev-MCU-analysisMapFile-4.png)

配置中需勾选上：Symbols

ImageSymbol Table：映射符号表，也就是各个段所存储对应地址的表（这一项比较重要）。

Symbols分为两大类

1.Local Symbols局部

2.Global Symbols全局


内容要点

1.SymbolName：符号名称

2.Value：存储对应的地址；

大家会发现有0x0800xxxx、0x2000xxxx这样的地址。

0x0800xxxx指存储在FLASH里面的代码、变量等。

0x2000xxxx指存储在内存RAM中的变量Data等。

3.OvType：符号对应的类型

符号类型大概有几种：Number、Section、ThumbCode、Data等；

细心的朋友会发现：全局、静态变量等位于0x2000xxxx的内存RAM中。

4.Size：存储大小

这个容易理解，我们怀疑内存溢出，可以查看代码存储大小来分析。

5.Object(Section)：段目标

这里一般指所在模块（所在源文件）。

### MemoryMap of the image：内存（映射）分布(重要)
![5](/images/2019-01-08-embededDev-MCU-analysisMapFile-5.png)

配置中需勾选上：MemoryMap



Memory Map of theimage：内存（映射）分布，内容相对较多，也是比较重要的一项。



Image Entry point :0x08000131：指映射入口地址。



Load Region LR_IROM1 (Base:0x08000000, Size: 0x000004cc, Max: 0x00080000,ABSOLUTE):

指加载区域位于LR_IROM1开始地址0x08000000，大小有0x000004cc，这块区域最大为0x00080000.



执行区域：

Execution Region ER_IROM1

Execution Region RW_IRAM1

这个区域，其实就是对应我们目标配置中的区域，如下如：

![6](/images/2019-01-08-embededDev-MCU-analysisMapFile-6.png)

内容要点

1.BaseAddr：存储地址

0x0800xxxxFLASH地址和0x2000xxxx内存RAM地址。



2.Size：存储大小



3.Type：类型

Data：数据类型

Code：代码类型

Zero：未初始化变量类型

PAD：这个类型在map文件中放在这个位置，其实它不能算这里的类型。要翻译的话，只能说的“补充类型”。

ARM处理器是32位的，如果定义一个8位或者16位变量就会剩余一部分，这里就是指的“补充”的那部分，会发现后面的其他几个选项都没有对应的值。



4.Attr：属性

RO：存储与ROM中的段

RW：存储与RAM中的段

5.SectionName：段名

这里也可以说为入口分类名，与第一章节“SectionCross References”指的模块、段一样。

大概包含：RESET、.ARM、 .text、 i、 .data、 .bss、 HEAP、 STACK等。

6.Object：目标

### Image component sizes：存储组成大小(重要)
![7](/images/2019-01-08-embededDev-MCU-analysisMapFile-7.png)

配置中需勾选上：SizeInfo

Image componentsizes：存储组成大小，其实主要就是对模块进行汇总存储大小信息。

这一章节内容相信大家都能理解，我们编译工程后，在编译窗口一般会看到类似如下一段信息：

Program Size: Code=908RO-data=320 RW-data=0 ZI-data=1024

Code：指代码的大小；

Ro-data：指除了内联数据(inlinedata)之外的常量数据；

RW-data：指可读写（RW）、已初始化的变量数据；

ZI-data：指未初始化（ZI）的变量数据；

Code、Ro-data：位于FLASH中；

RW-data、ZI-data：位于RAM中；

提醒：RW-data已初始化的数据会存储在Flash中，上电会从FLASH搬移至RAM中。

关系如下：

RO  Size = Code + RO Data

RW  Size = RW Data + ZI Data

ROM Size = Code + RO Data + RW Data


上面信息是比较全面的汇总，如果不想看那些模块的详细，只看汇总统计的信息可以在配置中只勾选“TotalsInfo”，对比信息：

![8](/images/2019-01-08-embededDev-MCU-analysisMapFile-8.png)

## map文件补充说明

- (.text)表示该section是存储的代码，对应的Ov Type是Thumb Code，对应的Attr是RO，也就是说存储在ROM中，也就是Flash中；
- (.constdata)表示该section是存储的常量，对应的Ov Tpye是Data，对应的Attr是RO，也就是说存储在ROM中，也就是Flash。
- (.data)表示该section存储的是RW数据，对应的Ov Tpye是Data，对应的Attr是RW，也就是存储在ROM中，也就是Flash,但是上电运行时，会将RW数据复制一份到RAM中，比如初始化的全局变量；
- (.bss)表示该section存储的是RW数据，但是对应的Type是Zero，也就是对应的Zi-data,比如未初始化的全局变量；



## 实验验证

一下实验结果均在keil 5中实验得出。

- 初始化的全局变量

如:u16 FALSH_ID=0; 注意初始化为0也是初始化。

在map文件：Image Symbol Table->Global Symbols中可以找到如下：
```

FALSH_ID                                 0x20000120   Data           2  main.o(.data)

```
其中main.o表示所在对象，(.data)表示存储段，也就说该变量(符号)是以RW的形式存储，也就是对应RW-data。

- 未初始化的全局变量

如：unsigned char RxBuffer[256];

在map文件：Image Symbol Table->Global Symbols中可以找到如下：
```
    RxBuffer                                 0x20000130   Data         256  main.o(.bss)
```

其中(.bss)，表示该变量（符号）是以ZI的形式存储，也就是对应zi-data.

- 初始化的局部变量数组

值得注意的是，初始化的局部变量数组会以.constdata存储，也就是RO-data，也就是存储在ROM(flash)中，这一点似乎和我们的认知有冲突，一般而言，我们认为所有的局部变量都应该是运行时创建的，怎么还会在flash中存储呢？这个问题我也没弄清楚。当然我也验证了如果不初始化的局部变量是不会在RW或RO中开辟空间的，我猜测是编译器优化的结果。

实验方法：在main()中添加：int  aaa[10]={1,2,3,4,5,6,7,8,9,0}; 后可以在map中找到新增的：

```
    0x08002ed4   0x00000028   Data   RO          408    .constdata          main.o
```
其中0x00000028为Size,也就是40byte,可见 section name 是.constdata。
而改成：int aaa[10]; 后就不会增加。

同样的我也测试了：int bbb=2; Ro-data并不会增加，仅仅是code会相应的增加。

同样的我也测试了：int const bbb=3; Ro-data并不会增加，仅仅是code会相应的增加。

- 常量全局变量

这个毫无疑问肯定是.constdata,也就是RO-data.

比如在全局作用域：int const ccc=3; Ro-data会增加4byte,同样的在map文件中也能找到这个变量。

- code的大小

code的大小是编译所有程序的代码量所占的空间，一般而言编译器都会做编译优化。所以手动写很简单的代码，去测试某条代码的指令会占用多少空间，是很难实现的。



## 参考阅读
- [Keil综合（03）_map文件全解析](https://blog.csdn.net/ybhuangfugui/article/details/75948282#rd)
