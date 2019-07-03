---
layout: post
title:  "STMCU中RAM和Flash的研究"
date:   2019-01-07 13:34:54
categories: 嵌入式开发
tags: 编整  MCU开发
author: wuxy
---

* content
{:toc}

## 几个问题
- STMCU的程序是在RAM还是Flash中运行的？
- STM32如何查看Flash和RAM使用空间以及如何压缩RAM的使用空间？
- “Program Size: Code=11980 RO-data=672 RW-data=14968 ZI-data=2072”分别是指什么？

## STM32中的程序在RAM还是FLASH里运行？

![STM32中的程序在RAM还是FLASH里运行](/images/2019-01-07-embededDev-MCU-RAMAndFlash-1.png)

## Program Size: Code=x RO-data=x RW-data=x ZI-data=x 的含义

Program Size: Code=x RO-data=x RW-data=x ZI-data=x 的含义

Code(代码): 程序所占用的FLASH大小，存储在FLASH.

RO-data(只读的数据): Read-only-data，程序定义的常量，如const型，存储在FLASH中。

RW-data(有初始值要求的、可读可写的数据):
Read-write-data,已经被初始化的变量，存储在FLASH中。初始化时RW-data从flash拷贝到SRAM。

ZI-data:Zero-Init-data,未被初始化的可读写变量，存储在SRAM中。ZI-data不会被算做代码里因为不会被初始化。

ROM(Flash) size = Code + RO-data + RW-data;
RAM size = RW-data + ZI-data
简单的说就是在烧写的时候是FLASH中的被占用的空间为：Code+RO Data+RW Data
程序运行的时候，芯片内部RAM使用的空间为： RW Data + ZI Data

```
Total RO  Size (Code + RO Data)                56284 (54.96kB[注：54.96kB = 56284/1024kB]，下同)
Total RW  Size (RW Data + ZI Data)              7424 (7.25kB)
Total ROM Size (Code + RO Data + RW Data)      56456 (55.13kB)
```
## 扩展阅读(不太重要)


### ROM 、RAM和FLASH 的区别

ROM和RAM指的都是半导体存储器，ROM是Read Only Memory的缩写，RAM是Random Access Memory的缩写。ROM在系统停止供电的时候仍然可以保持数据，而RAM通常都是在掉电之后就丢失数据，典型的RAM就是计算机的内存。



RAM 有两大类，一种称为静态RAM（Static RAM/SRAM），SRAM速度非常快，是目前读写最快的存储设备了，但是它也非常昂贵，所以只在要求很苛刻的地方使用，譬如CPU的一级缓冲，二级缓冲。另一种称为动态RAM（Dynamic RAM/DRAM），DRAM保留数据的时间很短，速度也比SRAM慢，不过它还是比任何的ROM都要快，但从价格上来说DRAM相比SRAM要便宜很多，计算机内存就是DRAM的。

DRAM分为很多种，常见的主要有FPRAM/FastPage、EDORAM、SDRAM、DDR RAM、RDRAM、SGRAM以及WRAM等，这里介绍其中的一种DDR RAM。

DDR RAM（Double-Date-Rate RAM）也称作DDR SDRAM，这种改进型的RAM和SDRAM是基本一样的，不同之处在于它可以在一个时钟读写两次数据，这样就使得数据传输速度加倍了。这是目前电脑中用得最多的内存，而且它有着成本优势，事实上击败了Intel的另外一种内存标准－Rambus DRAM。在很多高端的显卡上，也配备了高速DDR RAM来提高带宽，这可以大幅度提高3D加速卡的像素渲染能力。

内存工作原理　　

　　内存是用来存放当前正在使用的（即执行中）的数据和程序，我们平常所提到的计算机的内存指的是动态内存（即DRAM），动态内存中所谓的"动态"，指的是当我们将数据写入DRAM后，经过一段时间，数据会丢失，因此需要一个额外设电路进行内存刷新操作。

　　具体的工作过程是这样的：一个DRAM的存储单元存储的是0还是1取决于电容是否有电荷，有电荷代表1，无电荷代表0。但时间一长，代表1的电容会放电，代表0的电容会吸收电荷，这就是数据丢失的原因；刷新操作定期对电容进行检查，若电量大于满电量的1／2，则认为其代表1，并把电容充满电；若电量小于 1／2，则认为其代表0，并把电容放电，藉此来保持数据的连续性。



ROM也有很多种，PROM是可编程的ROM，PROM和EPROM （可擦除可编程ROM）两者区别是，PROM是一次性的，也就是软件灌入后，就无法修改了，这种是早期的产品，现在已经不可能使用了，而EPROM是通过紫外光的照射擦出原先的程序，是一种通用的存储器。另外一种EEPROM是通过电子擦除，价格很高，写入时间很长，写入很慢。

举个例子，手机软件一般放在EEPROM中，我们打电话，有些最后拨打的号码，暂时是存在SRAM中的，不是马上写入通过记录（通话记录保存在EEPROM中），因为当时有很重要工作（通话）要做，如果写入，漫长的等待是让用户忍无可忍的。



FLASH 存储器又称闪存，它结合了ROM和RAM的长处，不仅具备电子可擦除可编程（EEPROM）的性能，还不会断电丢失数据同时可以快速读取数据（NVRAM 的优势），U盘和MP3里用的就是这种存储器。在过去的20年里，嵌入式系统一直使用ROM（EPROM）作为它们的存储设备，然而近年来Flash全面代替了ROM（EPROM）在嵌入式系统中的地位，用作存储Bootloader以及操作系统或者程序代码或者直接当硬盘使用（U盘）。

目前Flash主要有两种NOR Flash和NADN Flash。

NOR Flash的读取和我们常见的SDRAM的读取是一样，用户可以直接运行装载在NOR FLASH里面的代码，这样可以减少SRAM的容量从而节约了成本。

NAND Flash没有采取内存的随机读取技术，它的读取是以一次读取一块的形式来进行的，通常是一次读取512个字节，采用这种技术的Flash比较廉价。用户不能直接运行NAND Flash上的代码，因此好多使用NAND Flash的开发板除了使用NAND Flah以外，还作上了一块小的NOR Flash来运行启动代码。

一般小容量的用NOR Flash，因为其读取速度快，多用来存储操作系统等重要信息，而大容量的用NAND FLASH，最常见的NAND FLASH应用是嵌入式系统采用的DOC（Disk On Chip）和我们通常用的"闪盘"，可以在线擦除。目前市面上的FLASH 主要来自Intel，AMD，Fujitsu和Toshiba，而生产NAND Flash的主要厂家有Samsung和Toshiba。



NAND Flash和NOR Flash的比较

NOR 和NAND是现在市场上两种主要的非易失闪存技术。Intel于1988年首先开发出NOR flash技术，彻底改变了原先由EPROM和EEPROM一统天下的局面。紧接着，1989年，东芝公司发表了NAND flash结构，强调降低每比特的成本，更高的性能，并且象磁盘一样可以通过接口轻松升级。但是经过了十多年之后，仍然有相当多的硬件工程师分不清NOR 和NAND闪存。　　

相"flash存储器"经常可以与相"NOR存储器"互换使用。许多业内人士也搞不清楚NAND闪存技术相对于NOR技术的优越之处，因为大多数情况下闪存只是用来存储少量的代码，这时NOR闪存更适合一些。而NAND则是高数据存储密度的理想解决方案。

NOR 是现在市场上主要的非易失闪存技术。NOR一般只用来存储少量的代码；NOR主要应用在代码存储介质中。NOR的特点是应用简单、无需专门的接口电路、传输效率高，它是属于芯片内执行(XIP, eXecute In Place)，这样应用程序可以直接在（NOR型）flash闪存内运行，不必再把代码读到系统RAM中。在1～4MB的小容量时具有很高的成本效益，但是很低的写入和擦除速度大大影响了它的性能。NOR flash带有SRAM接口，有足够的地址引脚来寻址，可以很容易地存取其内部的每一个字节。NOR flash占据了容量为1～16MB闪存市场的大部分。　　

NAND结构能提供极高的单元密度，可以达到高存储密度，并且写入和擦除的速度也很快。应用NAND的困难在于flash的管理和需要特殊的系统接口。

1、性能比较:　 　

flash闪存是非易失存储器，可以对称为块的存储器单元块进行擦写和再编程。任何flash器件的写入操作只能在空或已擦除的单元内进行，所以大多数情况下，在进行写入操作之前必须先执行擦除。NAND器件执行擦除操作是十分简单的，而NOR则要求在进行擦除前先要将目标块内所有的位都写为1。　　

由于擦除NOR器件时是以64～128KB的块进行的，执行一个写入/擦除操作的时间为5s，与此相反，擦除NAND器件是以8～32KB的块进行的，执行相同的操作最多只需要4ms。　　

执行擦除时块尺寸的不同进一步拉大了NOR和NADN之间的性能差距，统计表明，对于给定的一套写入操作(尤其是更新小文件时)，更多的擦除操作必须在基于NOR的单元中进行。这样，当选择存储解决方案时，设计师必须权衡以下的各项因素：　　

* NOR的读速度比NAND稍快一些。　　

* NAND的写入速度比NOR快很多。　　

* NAND的4ms擦除速度远比NOR的5s快。　　

* 大多数写入操作需要先进行擦除操作。　　

* NAND的擦除单元更小，相应的擦除电路更少。

(注：NOR FLASH SECTOR擦除时间视品牌、大小不同而不同，比如，4M FLASH，有的SECTOR擦除时间为60ms，而有的需要最大6s。)

2、接口差别:　　

NOR flash带有SRAM接口，有足够的地址引脚来寻址，可以很容易地存取其内部的每一个字节。　　

NAND器件使用复杂的I/O口来串行地存取数据，各个产品或厂商的方法可能各不相同。8个引脚用来传送控制、地址和数据信息。　　

NAND读和写操作采用512字节的块，这一点有点像硬盘管理此类操作，很自然地，基于NAND的存储器就可以取代硬盘或其他块设备。

3、容量和成本:　　

NAND flash的单元尺寸几乎是NOR器件的一半，由于生产过程更为简单，NAND结构可以在给定的模具尺寸内提供更高的容量，也就相应地降低了价格。　　

NOR flash占据了容量为1～16MB闪存市场的大部分，而NAND flash只是用在8～128MB的产品当中，这也说明NOR主要应用在代码存储介质中，NAND适合于数据存储，NAND在CompactFlash、 Secure Digital、PC Cards和MMC存储卡市场上所占份额最大。

4、可靠性和耐用性:　　

采用flahs介质时一个需要重点考虑的问题是可靠性。对于需要扩展MTBF的系统来说，Flash是非常合适的存储方案。可以从寿命(耐用性)、位交换和坏块处理三个方面来比较NOR和NAND的可靠性。　　

A) 寿命(耐用性)　　

在NAND闪存中每个块的最大擦写次数是一百万次，而NOR的擦写次数是十万次。NAND存储器除了具有10比1的块擦除周期优势，典型的NAND块尺寸要比NOR器件小8倍，每个NAND存储器块在给定的时间内的删除次数要少一些。　　

B) 位交换　　

所有flash器件都受位交换现象的困扰。在某些情况下(很少见，NAND发生的次数要比NOR多)，一个比特(bit)位会发生反转或被报告反转了。　　

一位的变化可能不很明显，但是如果发生在一个关键文件上，这个小小的故障可能导致系统停机。如果只是报告有问题，多读几次就可能解决了。　　

当然，如果这个位真的改变了，就必须采用错误探测/错误更正(EDC/ECC)算法。位反转的问题更多见于NAND闪存，NAND的供应商建议使用NAND闪存的时候，同时使用EDC/ECC算法。　　

这个问题对于用NAND存储多媒体信息时倒不是致命的。当然，如果用本地存储设备来存储操作系统、配置文件或其他敏感信息时，必须使用EDC/ECC系统以确保可靠性。　　

C) 坏块处理　　

NAND器件中的坏块是随机分布的。以前也曾有过消除坏块的努力，但发现成品率太低，代价太高，根本不划算。　　

NAND器件需要对介质进行初始化扫描以发现坏块，并将坏块标记为不可用。在已制成的器件中，如果通过可靠的方法不能进行这项处理，将导致高故障率。

5、易于使用:　　

可以非常直接地使用基于NOR的闪存，可以像其他存储器那样连接，并可以在上面直接运行代码。　　

由于需要I/O接口，NAND要复杂得多。各种NAND器件的存取方法因厂家而异。　　

在使用NAND器件时，必须先写入驱动程序，才能继续执行其他操作。向NAND器件写入信息需要相当的技巧，因为设计师绝不能向坏块写入，这就意味着在NAND器件上自始至终都必须进行虚拟映射。

6、软件支持:　　

当讨论软件支持的时候，应该区别基本的读/写/擦操作和高一级的用于磁盘仿真和闪存管理算法的软件,包括性能优化。　　

在NOR器件上运行代码不需要任何的软件支持，在NAND器件上进行同样操作时，通常需要驱动程序，也就是内存技术驱动程序(MTD)，NAND和NOR器件在进行写入和擦除操作时都需要MTD。　　

使用NOR器件时所需要的MTD要相对少一些，许多厂商都提供用于NOR器件的更高级软件，这其中包括M-System的TrueFFS驱动，该驱动被 Wind River System、Microsoft、QNX Software System、Symbian和Intel等厂商所采用。

驱动还用于对DiskOnChip产品进行仿真和NAND闪存的管理，包括纠错、坏块处理和损耗平衡。

NOR FLASH的主要供应商是INTEL ,MICRO等厂商，曾经是FLASH的主流产品，但现在被NAND FLASH挤的比较难受。它的优点是可以直接从FLASH中运行程序，但是工艺复杂，价格比较贵。

NAND FLASH的主要供应商是SAMSUNG和东芝，在U盘、各种存储卡、MP3播放器里面的都是这种FLASH，由于工艺上的不同，它比NOR FLASH拥有更大存储容量，而且便宜。但也有缺点，就是无法寻址直接运行程序，只能存储数据。另外NAND FLASH 非常容易出现坏区，所以需要有校验的算法。

在掌上电脑里要使用NAND FLASH 存储数据和程序，但是必须有NOR FLASH来启动。除了SAMSUNG处理器，其他用在掌上电脑的主流处理器还不支持直接由NAND FLASH 启动程序。因此，必须先用一片小的NOR FLASH 启动机器，在把OS等软件从NAND FLASH 载入SDRAM中运行才行，挺麻烦的。



DRAM 利用MOS管的栅电容上的电荷来存储信息，一旦掉电信息会全部的丢失，由于栅极会漏电，所以每隔一定的时间就需要一个刷新机构给这些栅电容补充电荷，并且每读出一次数据之后也需要补充电荷，这个就叫动态刷新，所以称其为动态随机存储器。由于它只使用一个MOS管来存信息，所以集成度可以很高，容量能够做的很大。SDRAM比它多了一个与CPU时钟同步。

SRAM 利用寄存器来存储信息，所以一旦掉电，资料就会全部丢失，只要供电，它的资料就会一直存在，不需要动态刷新，所以叫静态随机存储器。

以上主要用于系统内存储器，容量大，不需要断电后仍保存数据的。

Flash ROM 是利用浮置栅上的电容存储电荷来保存信息，因为浮置栅不会漏电，所以断电后信息仍然可以保存。也由于其机构简单所以集成度可以做的很高，容量可以很大。 Flash rom写入前需要用电进行擦除，而且擦除不同，EEPROM可以以byte(字节)为单位进行，flash rom只能以sector(扇区)为单位进行。不过其写入时可以byte为单位。flash rom主要用于bios，U盘，Mp3等需要大容量且断电不丢数据的设备。

## 参考阅读
- [STM32如何查看Flash和RAM使用空间以及如何压缩RAM的使用空间](https://blog.csdn.net/jdsnpgxj/article/details/78605341)
- [STM32中的程序在RAM还是FLASH里运行？](https://blog.csdn.net/yangkuiwu/article/details/78219995)
- [Keil 中 Program Size: Code RO-data RW-data ZI-data 所代表的意思](https://www.cnblogs.com/xidongs/p/5771798.html)
