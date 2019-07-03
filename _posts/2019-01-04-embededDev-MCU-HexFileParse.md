---
layout: post
title:  "Hex文件解析研究(含源码)"
date:   2019-01-04 10:04:54
categories: 嵌入式开发
tags: 编整  MCU开发
author: wuxy
---

* content
{:toc}

## 课题背景

之所以对Hex文件格式其了研究的想法，是因为想造轮子了，想手动编码实现下载hex文件当MCU中，而不是通过官方/非官方的下载软件，如：keil,Jtag,flyMCU等，虽然，那些软件确实已经能够解决基本的需求，但是想要深入定制下载器，还是不够的。

比如，我想通过无线的形式传输Hex文件，而Hex文件却是二进制文件，很难直接通过无线模块传输，而必须转化为16进制字符。而在发送和接收时都需要解析和反解析。

比如，在手动编码实现SWD协议时，数据也是通过字符的形式发送到目标板上的。

本文介绍了2种Hex文件，ARM-MCU和Intel的，我亲测了ARM-MCU的，但是基本格式差不多。

## HEX文件格式详解-MCU

参考链接：http://www.forwhat.cn/post-240.html，另外在CSDN上也能搜索到很多，本文整理了比较完善的。

Hex文件是可以烧录到MCU中，被MCU执行的一种文件格式。如果用记事本打开可发现，整个文件以行为单位，每行以冒号开头，内容全部为16进制码（以ASCII码形式显示）。Hex文件可以按照如下的方式进行拆分来分析其中的内容：

例如 “:1000080080318B1E0828092820280B1D0C280D2854”可以被看作“0x10 0x00 0x08 0x00 0x80 0x31 0x8B 0x1E 0x08 0x28 0x09 0x28 0x20 0x28 0x0B 0x1D 0x0C 0x28 0x0D 0x28 0x54”

第一个字节 0x10表示本行数据的长度；

第二、三字节 0x00 0x08表示本行数据的起始地址；

第四字节 0x00表示数据类型，数据类型有：0x00、0x01、0x02、0x03、0x04、0x05。

'00' Data Rrecord：用来记录数据，HEX文件的大部分记录都是数据记录

'01' End of File Record: 用来标识文件结束，放在文件的最后，标识HEX文件的结尾

'02' Extended Segment Address Record: 用来标识扩展段地址的记录

'03' Start Segment Address Record:开始段地址记录

'04' Extended Linear Address Record: 用来标识扩展线性地址的记录

'05' Start Linear Address Record:开始线性地址记录

然后是数据，最后一个字节 0x54为校验和。

校验和的算法为：计算0x54前所有16进制码的累加和(不计进位)，检验和 = 0x100 - 累加和

在上面的后2种记录，都是用来提供地址信息的。每次碰到这2个记录的时候，都可以根据记录计算出一个“基”地址。对于后面的数据记录，计算地址的时候，都是以这些“基”地址为基础的。

HEX文件都是由记录（RECORD）组成的。在HEX文件里面，每一行代表一个记录。记录的基本格式为：


Record mark ‘:’

Length

Load offset

Record type

INFO or DATA

CHKSUM

1 byte

1 byte

2 bytes

1 byte

n bytes

1 byte

看个例子：

:020000040008F2

:10000400FF00A0E314209FE5001092E5011092E5A3

:00000001FF      

对上面的HEX文件进行分析：

第1条记录的长度为02，LOAD OFFSET为0000，RECTYPE为04，说明该记录为扩展段地址记录。数据为0008，校验和为F2。从这个记录的长度和数据，我们可以计算出一个基地址，这个地址为(0x0008 << 16)。后面的数据记录都以这个地址为基地址。

第2条记录的长度为10（16），LOAD OFFSET为0004，RECTYPE为00，说明该记录为数据记录。数据为FF00A0E314209FE5001092E5011092E5，共16个BYTE。这个记录的校验和为A3。此时的基地址为0X80000，加上OFFSET，这个记录里的16BYTE的数据的起始地址就是0x80000 + 0x0004 = 0x80004.

第3条记录的长度为00，LOAD OFFSET为0000，TYPE ＝ 01，校验和为FF。说明这个是一个END OF FILE RECORD，标识文件的结尾。

在上面这个例子里，实际的数据只有16个BYTE：FF00A0E314209FE5001092E5011092E5，其起始地址为0x0004.

其实文件格式很简单，而目标板真正需要的只有实际数据，根据上面的格式说明，我编写了一个小程序，用于直接由Hex文件生成C代码数组文件。本人亲测有效，如有问题可以邮件联系我。

```

#run by python3.6 in IDLE
#author:wuxy
#date:2019-01-04
#function:将hex文件中有效数据提取到C数组文件。

size=0
def parseHexPerLine(str):
    global size
    if len(str)<1:
        return
    if str[0]!=':':
        print('the first char is not :')
        return
    dataLen=int(str[1:3],16)*2 #数据长度,实际是字符串的长度
   # print('dataLen: ',dataLen)

    if str[7:9]=='00': #类型为数据
        size+=dataLen
        dataStr=str[9:9+dataLen]
        parseStr=''
        for i in range(0,dataLen,2):
            parseStr+='0x'
            parseStr+=dataStr[i:i+2]
            parseStr+=','

        parseStr+='\n'   #为了排版需要
        return parseStr
    else:
        return ''

fileName=input('please input hex file name: ')
cArrayFile=open('aArrayFile.c','w')
cArrayFile.write('const unsigned char ')
cArrayFile.write(fileName)
cArrayFile.write('[]={\n')

for line in open(fileName):
    print(line)
    str=parseHexPerLine(line)
    print(str)
    cArrayFile.write(str)

cArrayFile.seek(cArrayFile.tell()-3) #剔除最后一个逗号,本来应该是-1，但是由于前面加了一个'\n'.
cArrayFile.write('};')
cArrayFile.close()
size/=2
print('size: ',size)

```

其中，size是用于记录数组的大小。

写的比较简陋，但是很实用。


## HEX文件格式解析-Intel

Intel HEX 文件是由一行行符合Intel HEX 文件格式的文本所 构 成的ASCII 文本文件。在Intel HEX 文件中，每一行包含一 个 HEX 记录 。 这 些 记录 由 对应 机器 语 言 码 和/ 或常量 数 据的十六 进 制 编码数 字 组 成。Intel HEX 文件通常用于 传输将 被存于ROM 或者EPROM 中的程序和 数 据。大多 数 EPROM 编 程器或模 拟器使用Intel HEX 文件。

Hex文件是可以烧写到单片机中，被单片机执行的一种文件格式，生成Hex文件的方式由很多种，可以通过不同的编译器将C程序或者汇编程序编译生成hex。

一般Hex文件通过记事本就可以打开。可以发现一般Hex文件的记录格式如下：
![Hex记录格式](/images/2019-01-04-embededDev-MCU-HexFileParse-1.png)

Intel HEX 由任意数量的十六 进 制 记录组 成。每 个记录 包含5 个 域， 它们按以下格式排列：

每一组字母 对应 一 个 不同的域，每一 个 字母 对应 一 个 十六 进 制 编码 的 数 字。每一 个 域由至少 两个 十六 进制 编码数 字 组 成， 它们构 成一 个 字 节 ，就像以下描述的那 样：

：(冒号)每个Intel HEX 记录 都由冒 号开头 ；
LL 是 数 据 长 度域, 它 代表 记录当 中 数 据字 节 (dd) 的 数量 ；
aaaa 是地址域, 它代表 记录当 中 数据的起始地址；
TT是代表HEX 记录类 型的域 , 它 可能是以下 数 据 当 中的一 个：
    00 – 数 据 记录（Data Record）
    01 – 文件结 束 记录（End of FileRecord）
    02 – 扩展段地址 记录（ExtendedSegment Address Record）

03 – 开始段地址 记录（Start Segment Address Record）
    04 – 扩展 线 性地址 记录（Extended Linear Address Record）

05 – 开始线性地址 记录（Extended Segment Address Record）
dd 是数 据域 , 它 代表一 个 字 节 的 数 据. 一 个记录 可以有 许 多 数 据字 节 . 记录当 中 数 据字 节 的 数 量必 须 和数 据 长 度域(ll) 中指定的 数字相符.
cc 是校验 和域 , 它 表示 这个记录 的校 验 和. 校 验 和的 计 算是通 过将记录当 中所有十六 进 制 编码数 字 对 的 值相加, 以256 为 模 进 行以下 补 足.

表示为：“：[1字节长度][2字节地址][1字节记录类型][n字节数据段][1字节校验和] ”



具体根据记录类型分析如下：

(1)数据记录”00”

Intel HEX文件由任意数 量以回车换行符结束的数据记录组成数据记录外观如下:
    :10246200464C5549442050524F46494C4500464C33
其中:10 是这个记录当中 数 据字 节 的 数量.即0x10 ；
     2462 是数据 将 被下 载 到存 储 器 当中的地址.即0x2462 ；

00 是记录类型( 数 据 记录).即0x00 ；
464C…464C是 数据.分别代表0x46,0x4C... ；
33 是这个记录的校 验和即0x33；计算方法如下：256D-(10H+24H+62H+00H+46H+4CH+55H+49H+44H+20H+50H+52H+4FH+46H+49H+4CH+45H+00H+46H+4CH)/100H=33H；

(2)文件结束(EOF)”01”

Intel HEX文件必须以文件结束(EOF) 记录结束这个记录的记录类的值必须是01.EOF 记录 外 观总是如下:
   :00000001FF
其中:00 是记录当中 数 据字 节 的 数量.
   0000 是数据被下载到存储器当中的地址. 在文件结束记录当中地址是没有意义，被忽略的.0000h 是典型的地址；
   01 是记录类型 01( 文件 结 束 记录)
   FF 是 这个记录 的校 验 和, 计算方法如下: 256D-（00H+00H+00H+01H）=FFH；

(3)扩展线性地址记录(HEX386) ”04”

由于每行标识数据地址的只有2Byte，所以最大只能到64K，为了可以保存高地址的数据，就有了Extended Linear AddressRecord。如果这行的数据类型是0x04，那么，这行的数据就是随后数据的基地址。

扩展线性地址记录也叫作32位地址记录或HEX386记录.这些记录含数据的高16位扩展线性地址记录总是有两个数据字节，外观如下：
    :02000004FFFFFC

其中:02 是这个记录当中 数 据字 节 的 数量.
0000 是地址域, 对于 扩 展 线 性地址 记录 , 这个 域 总是0000.
04 是记录类型 04( 扩 展 线 性地址 记录)
FFFF 是地址的高16 位.
FC 是这个记录的校 验 和, 计算如下: 256D-（02H+00H+00H+04H+FFH+FFH）/100H=FFH；

当一 个扩展 线 性地址记录被读 取, 存 储于数据域的扩展线性地址被保存，它被应于

从 Intel HEX 文件 读取 来 的 随 后的 记录 . 线 性地址保持有效, 到 它 被另外一 个扩址记录 所改 变。

通 过 把 记录当 中的地址域 与 被移位的 来 自 扩 展 线 性地址 记录 的地址 数 据相加

 获 得 数 据 记录 的 绝对 存 储器地址。

以下的例子演示了这个过 程:

:0200000480007A    //数据记录的绝对存储器地址高16位为0x8000               

:100000001D000A00000000000000000000000000C9

:100010000000000085F170706F0104005D00BD00FC

第一行，是Extended Linear Address Record，里面的数据，也就是基地址是0x8000，第二行是DataRecord，里面的地址值是0x0000。那么数据1D000A00000000000000000000000000（共16个字节）要写入FLASH中的地址为 (0x8000<< 16)| 0x0000，也就是写入FLASH的0x80000000这个地址；第三行的数据写入地址为0x80000010.当一个HEX文件的数据超过64k的时候，文件中就会出现多个Extended Linear Address Record。

(4)扩展段地址记录(HEX86)“02“

扩展段地址记录也叫HEX86 记录 , 它包括4-19 位数据地址段. 扩展段地址记总是有两

个数 据字节 , 外观如下:
:020000021200EA
其中:02 是记录当中 数 据字 节 的 数量；
0000 是地址域. 对于 扩 展段地址 记录 , 这个 域 总是0000；
02 是记录类型 02( 扩 展段地址 记录)；
1200 是地址段；
EA 是这个记录的校 验 和；

当一 个扩 展段地址 记录 被 读 取, 存 储 于 数 据域的 扩 展段地址被保存, 它 被 应 用于 从 Intel HEX 文件 读 取 来的 随 后的 记录 . 段地址保持有效, 直到 它 被另外一 个扩 展地址 记录 所改 变。

通 过 把 记录当 中的地址域 与 被移位的 来 自 扩 展段地址 记录 的地址 数 据相加 获 得 数 据 记录 的 绝对 存 储器地址。
    以下的例子演示了这个过 程..
来自 数 据 记录地址域的地址          2462
扩展段地址 记录数据域             +  1200
                               ---------
绝对存 储 器地址                   00014462


Intel HEX 文件例子:
下面是一个 完整的Intel HEX 文件的例子:
:10001300AC12AD13AE10AF1112002F8E0E8F0F2244
:10000300E50B250DF509E50A350CF5081200132259
:03000000020023D8
:0C002300787FE4F6D8FD7581130200031D
:10002F00EFF88DF0A4FFEDC5F0CEA42EFEEC88F016
:04003F00A42EFE22CB
:00000001FF

贴一段解析的c语言，这才是精华：改代码是解析intel Hex的数据格式，和MCU的不太一样，但是可以参考，由于没有intel hex文件，暂时没有实测。

```
int ParseIHexPerLine(const char *buf,const char *path,int line)
{
	unsigned int nbytes=0,addr=0,type=0,i,val,line_chksum;
	unsigned char data[1024];
	unsigned char cksum;
	const char *s=buf;			
	if(*s!=':') //第一个为冒号
	{  
		fprintf(stderr,"%s:%s: format violation (1)/n",path,line);
		return(1);  
	}
	++s;
	//接下来的8个字节为数据大小、地址等
	if(sscanf(s,"%02x%04x%02x",&nbytes,&addr,&type)!=3)
	{  
		fprintf(stderr,"%s:%s: format violation (2)/n",path,line);
		return(1);  
	}
	s+=8;
	//读到的类型
	if(type==0) //为数据段
	{
		if(!(nbytes>=0 && nbytes<1024))
		{
			perror("nbyte per line unsupport/n");
			return(-1);
		}

		cksum=nbytes+addr+(addr>>8)+type;
		//
		for(i=0; i<nbytes; i++)
		{
			val=0;
			if(sscanf(s,"%02x",&val)!=1)
			{  
				fprintf(stderr,"%s:%s: format violation (3)/n",path,line);
				return(1);
			}
			s+=2;
			data[i]=val;
			cksum+=val;
		}
		//
		line_chksum=0;
		if(sscanf(s,"%02x",&line_chksum)!=1)
		{  
			fprintf(stderr,"%s:%s: format violation (4)/n",path,line);
			return(1);  
		}
		if((cksum+line_chksum)&0xff)
		{  
			fprintf(stderr,"%s:%s: checksum mismatch (%u/%u)/n",
					cksum,line_chksum);  
			return(1);  
		}
		if(WriteRAM(addr,data,nbytes))
			return(1);  
	}
	else if(type==1)
	{
		// EOF marker. Oh well, trust it.
		return(-1);
	}
	else
	{
		fprintf(stderr,"%s:%s: Unknown entry type %d/n",type);
		return(1);
	}
	return(0);
}


```

代码分析：

- 简单的将hex文件中的一行有效数据，保存在data[]中，不包含地址，类型的信息；
- 通过函数WriteRAM()将数据写进RAM中，这部分函数没给出具体实现；
- 调用时，调用一次仅仅是解析hex文件的一行。
