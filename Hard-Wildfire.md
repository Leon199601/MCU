# 野火F103MINI板

安装各种芯片包，才能使用对应的芯片进行调试（KEIL5）

## DAP仿真器下载程序

### 仿真器配置

1、Debug的选配

2、Utilities的选配

3、Debug Settings的选配

### 选目标板

### 下载程序

Application running ...

## 串口下载程序（使用相对于的软件）

使用的是hex文件或者bin文件来进行烧录程序

## STM32

其中分为软件部分和硬件部分

软件部分使用的是keil5软件、VS2019、vcode等等，多学习c语言的编程知识，主要的是逻辑思维，多看不断积累，不断学习

硬件部分使用的是AD软件，来进行原理图的设计，分析电源部分，产生的各种问题，如出现辐射现象，要考虑节约成本，多方面考虑问题，不断积累，很吃经验。经验告诉自己固定这样做。并且进行PCB的布局之类的，建议看哔哩哔哩中的凡亿教育的课程，多学习AD软件的快捷键。

## 寄存器

### 芯片

芯片包括M3内核和外设连接构成

### 存储器映射

4GB的地址空间 = 8 * 512 MB = 8 * 512 * 1024 * 1024 Bytes = 4294967296 Bytes

4294967296（十进制）= 0x100000000（十六进制）地址范围：0x00000000~0xFFFFFFFF

平分为8个地址

536870912（十进制） = 0x20000000（十六进制）地址范围：0x00000000~0x1FFFFFFF

1个字节 = 1 Byte = 8 Bits

1 KB = 1024 Bytes（表示1024个地址，1024（十进制） = 0x400（十六进制）地址范围：0x00~0x3FF）

1 MB = 1024 KB

1 GB = 1024 MB

Block0主要用于设计片内的FLASH，0x08000000~0x0807FFFF(512KB)

Block1用于设计片内的SRAM，0x20000000~0x2000FFFF（64KB）

### 寄存器映射

1个单元 = 4个字节 = 32 bits

eg：

​	GPIOB端口输出数据寄存器ODR的地址是 0x40010C0C，ODR寄存器是32bit，低16bit有效，对应16个外部IO

```c
// GPIOB 端口全部输出 高电平
*(unsigned int*)(0x4001 0C0C) = 0xFFFF;
```

0x4001 0C0C 在我们看来是GPIOB 端口ODR 的地址，但是在编译器看来，这只是一个普通的变量，是一个立即数，要想让编译器也认为是指针，我们得进行强制类型转换，把它转换成指针，即(unsigned int *)0x4001 0C0C，然后再对这个指针进行 * 操作。

```c
// GPIOB 端口全部输出 高电平
#define GPIOB_ODR *(unsigned int*)(GPIOB_BASE+0x0C)
GPIOB_ODR = 0xFF;
```

#### STM32外设地址映射

片上外设分为三条总线，APB1挂载低速外设，APB2和AHB挂载高速外设。

##### 总线基地址

| 总线名称 | 总线基地址  | 相对外设基地址的偏移 |
| -------- | ----------- | -------------------- |
| APB1     | 0x4000 0000 | 0x0                  |
| APB2     | 0x4001 0000 | 0x0001 0000          |
| AHB      | 0x4001 8000 | 0x0001 8000          |

##### 外设基地址

GPIO这个外设属于高速的外设，挂载到APB2总线上。

| 外设名称 | 外设基地址  | 相对APB2总线的地址偏移 |
| -------- | ----------- | ---------------------- |
| GPIOA    | 0x4001 0800 | 0x0000 0800            |
| GPIOB    | 0x4001 0C00 | 0x0000 0C00            |
| GPIOC    | 0x4001 1000 | 0x0000 1000            |
| GPIOD    | 0x4001 1400 | 0x0000 1400            |
| GPIOE    | 0x4001 1800 | 0x0000 1800            |
| GPIOF    | 0x4001 1C00 | 0x0000 1C00            |
| GPIOG    | 0x4001 2000 | 0x0000 2000            |

##### 外设寄存器

| 寄存器名称 | 寄存器地址  | 相对GPIO基址的偏移 |
| ---------- | ----------- | ------------------ |
| GPIOB_CRL  | 0x4001 0C00 | 0x00               |
| GPIOB_CRH  | 0x4001 0C04 | 0x04               |
| GPIOB_IDR  | 0x4001 0C08 | 0x08               |
| GPIOB_ODR  | 0x4001 0C0C | 0x0C               |
| GPIOB_BSRR | 0x4001 0C10 | 0x10               |
| GPIOB_BRR  | 0x4001 0C14 | 0x14               |
| GPIOB_LCKR | 0x4001 0C18 | 0x18               |

每一个寄存器都有16个引脚，分为三种状态：w、r、w\r，并包括设置或者清除位。eg：BR0写入“1”代表第0引脚输出“低电平”，但是写入“0”不会影响0位，BS0写入“1”代表引脚输出“高电平”。

#### 修改寄存器的位操作

##### 将变量的某位清零

```c
//定义一个变量a = 1001 1111 b (二进制数)
unsigned char a = 0x9f;

//对bit2 清零

a &= ~(1<<2);

//括号中的1 左移两位，(1<<2)得二进制数：0000 0100 b
//按位取反，~(1<<2)得1111 1011 b
//假如a 中原来的值为二进制数： a = 1001 1111 b
//所得的数与a 作”位与&”运算，a = (1001 1111 b)&(1111 1011 b),
//经过运算后，a 的值 a=1001 1011 b
//a的bit2位被清零，而其他位不变。
```

##### 将变量的某几个连续位清零

```c
//若把a 中的二进制位分成2 个一组
//即bit0、bit1 为第0 组，bit2、bit3 为第1 组，
// bit4、bit5 为第2 组，bit6、bit7 为第3 组
//要对第1 组的bit2、bit3 清零

a &= ~(3<<2*1);

//括号中的3 左移两位，(3<<2*1)得二进制数：0000 1100 b
//按位取反，~(3<<2*1)得1111 0011 b
//假如a 中原来的值为二进制数： a = 1001 1111 b
//所得的数与a 作”位与&”运算，a = (1001 1111 b)&(1111 0011 b),
//经过运算后，a 的值 a=1001 0011 b
// a 的第1 组的bit2、bit3 被清零，而其它位不变。

//上述(~(3<<2*1))中的(1)即为组编号;如清零第3 组bit6、bit7 此处应为3
//括号中的(2)为每组的位数，每组有2 个二进制位;若分成4 个一组，此处即为4
//括号中的(3)是组内所有位都为1 时的值;若分成4 个一组，此处即为二进制数“1111 b”

//例如对第2 组bit4、bit5 清零
a &= ~(3<<2*2);
```

##### 将变量的某几位进行赋值

```c
//a = 1000 0011 b
//此时对清零后的第2 组bit4、bit5 设置成二进制数“01 b ”

a |= (1<<2*2);
//a = 1001 0011 b，成功设置了第2 组的值，其它组不变
```

##### 将变量的某位取反

```c
//a = 1001 0011 b
//把bit6 取反，其它位不变

a ^=(1<<6);
//a = 1101 0011 b
```

## 新建工程---寄存器版

### 新建工程

#### 新建本地工程文件夹

为了工程目录更加清晰，我们在本地电脑上新建1 个文件夹用于存放整个工程，如命名为“LED”，然后在该目录下新建2 个文件夹

| 名称                  | 作用                                                |
| --------------------- | --------------------------------------------------- |
| Listing               | 存放编译器编译时候产生的c/汇编/链接的列表清单       |
| Output（或者Objects） | 存放编译产生的调试信息、hex文件、预览信息、封装库等 |

| 名称                  | 作用                                                |
| --------------------- | --------------------------------------------------- |
| LED                   | 存放startup_stm32f10x_hd.s、stm32f10x.h、main.c文件 |
| Listing               | 暂时为空                                            |
| Output（或者Objects） | 暂时为空                                            |

#### 新建工程

打开KEIL5，新建---Project---New Project...

选择CPU型号

添加文件---startup_stm32f10x_hd.s、stm32f10x.h、main.c

配置魔术棒选项卡---47页

下载器配置

### 下载程序

点击LOAD按钮

## 寄存器点亮LED灯

### GPIO

GPIOA到FPIOE共5组，每组16个引脚，具有输入输出功能

输入功能是检测外部输入电平，通过电平高低区分按键是否被按下

输出功能是高低电平的输出切换，实现开关控制，来控制灯的亮灭

#### 基本结构

![1](https://github.com/Leon199601/MCU/blob/main/pic/w-1.jpg)

##### 1、保护二极管及上、下拉电阻

当引脚电压高于Vdd时，上方二极管导通

当引脚电压小于Vss时，下方二极管导通，防止不正常电压引入芯片导致芯片烧毁，保护芯片。

如果直接驱动电机，必须加大功率及隔离电路驱动。

##### 2、P-MOS和N-MOS管

输出控制部分：该结构使得GPIO具有“推挽输出”和“开漏输出”

推挽输出，是根据两个MOS管的工作方式来命名。

输入高点平时，上方P-MOS导通，下方N-MOS关闭，对外输出高电平，高电平为3.3伏；

输入低点平时，下方N-MOS导通，上方P-MOS关闭，对外输出低电平，低电平为0伏；

推挽输出一般应用在输出电平为0和3.3伏而且需要高速切换开关状态的场合。STM32应用中，除了必须用开楼模式的场合，都习惯用推挽输出模式。

![2](https://github.com/Leon199601/MCU/blob/main/pic/w-2.jpg)

开漏输出，上方的P-MOS管完全不工作。

如果控制输出为0，低电平，则P-MOS管关闭，N-MOS管导通，输出接地；

如控制输出为1，则P-MOS和N-MOS都关闭，引脚既不输出高电平，也不输出低电平，为高阻态。正常使用必须外部接上拉电阻，等效电路，具有“线与”特性，也就是说，若有很多个开漏输出引脚连接在一起，只有当所有的引脚为高阻态，才由上拉电阻提供高电平，此高电平的电压为外部上拉电阻所接的电源的电压。若其中一个引脚为低电平，那线路相当于短路接地，整条线路都是低电平，0伏。

![3](https://github.com/Leon199601/MCU/blob/main/pic/w-3.jpg)

开漏输出一般应用在I2C、SMBUS通讯等需要“线与”功能的总线电路中。

除此之外，还用在电平不匹配的场合，如需要5伏的高电平，就可以外部接一个上拉电阻，上拉电源为5伏，GPIO设置为开漏模式，当输出高阻态时，由上拉电阻和电源向外输出5伏的电平。

![4](https://github.com/Leon199601/MCU/blob/main/pic/w-4.jpg)

##### 3、输出数据寄存器GPIOx_ODR

双MOS管结构电路的输入信号，是由GPIO“输出数据寄存器GPIOx_ODR"来控制，

“置位/复位寄存器GPIOx_BSRR"可以修改输出数据寄存器的值。

##### 4、复用功能输出

“复用”是指STM32的其他片上外设对GPIO引脚进行控制，从其他外设引出的“复用功能输出信号”与GPIO的数据寄存器，通过梯形结构作为开关切换选择。

例如：使用USART串口通讯时，需要用到某个GPIO引脚作为通讯发送引脚，这时候就可以把该GPIO引脚配置为USART串口复用功能，由串口外设控制该引脚，发送数据。

```c
//GPIOB  16个IO全部输出  0XFF
GPIOB-> = 0XFF;
```

##### 5、输入数据寄存器GPIOx_IDR

GPIO引脚经过内部的上、下拉电阻，可配置成上/下拉输入，再连接施密特触发器，信号经过触发器，模拟信号转化为0、1数字信号，存储在“输入数据寄存器GPIOx_IDR"，读取寄存器就可以了解GPIO引脚的电平状态。

```c
//读取GPIOB端口的16位数据值
uint16_t temp;
temp = GPIOB->IDR;
```

##### 6、复用功能输入

与复用功能输出类似，GPIO引脚的信号传输到STM32其他片上外设，由该外设读取引脚状态。

例如：使用USART串口通讯时，需要用到某个GPIO引脚作为通讯接收引脚，这时候就可以把该GPIO引脚配置为USART串口复用功能，由串口外设控制该引脚，接收数据。

##### 7、模拟输入输出

当GPIO 引脚用于ADC 采集电压的输入通道时，用作“模拟输入”功能，此时信号是不经过施密特触发器的，因为经过施密特触发器后信号只有0、1 两种状态，所以ADC 外设要采集到原始的模拟信号，信号源输入必须在施密特触发器之前。

类似地，当GPIO 引脚用于DAC 作为模拟电压输出通道时，此时作为“模拟输出”功能，DAC 的模拟信号输出就不经过双MOS 管结构，模拟信号直接输出到引脚。

#### GPIO工作模式

```c
//GPIOG 8种工作模式
typedef enum
{
	GPIO_Mode_AIN = 0x0,			//模拟输入
	GPIO_Mode_IN_FLOATING = 0x04,	//浮空输入
	GPIO_Mode_IPD = 0x28,			//下拉输入
	GPIO_Mode_IPU = 0x48,			//上拉输入
	GPIO_Mode_Out_OD = 0x14,		//开漏输出
	GPIO_Mode_Out_PP = 0x10,		//推挽输出
	GPIO_Mode_AF_OD = 0x1C,			//复用开漏输出
	GPIO_Mode_AF_PP = 0x18			//复用推挽输出
} GPIOMode_TypeDef;
```

8种工作模式大致分为三类：

##### 1、输入模式（模拟/浮空/上拉/下拉）

在输入模式时，施密特触发器打开，输出被禁止，可通过输入数据寄存器GPIOx_IDR读取I/O 状态。

输入模式，可设置为上拉、下拉、浮空和模拟输入四种。

上拉和下拉输入很好理解，默认的电平由上拉或者下拉决定。

浮空输入的电平是不确定的，完全由外部的输入决定，一般接按键的时候用的是这个模式。

模拟输入则用于ADC 采集。

##### 2、输出模式（推挽/开漏）

在输出模式中，推挽模式时双MOS 管以轮流方式工作，输出数据寄存器GPIOx_ODR可控制I/O 输出高低电平。开漏模式时，只有N-MOS 管工作，输出数据寄存器可控制I/O输出高阻态或低电平。输出速度可配置，有2MHz\10MHz\50MHz 的选项。此处的输出速度即I/O 支持的高低电平状态最高切换频率，支持的频率越高，功耗越大，如果功耗要求不严格，把速度设置成最大即可。
在输出模式时施密特触发器是打开的，即输入可用，通过输入数据寄存器GPIOx_IDR可读取I/O 的实际状态。

##### 3、复用功能（推挽/开漏）

复用功能模式中，输出使能，输出速度可配置，可工作在开漏及推挽模式，但是输出信号源于其它外设，输出数据寄存器GPIOx_ODR 无效；输入可用，通过输入数据寄存器可获取I/O 实际状态，但一般直接用外设的寄存器来获取该数据信号。
通过对GPIO 寄存器写入不同的参数，就可以改变GPIO 的工作模式，再强调一下，要了解具体寄存器时一定要查阅==《STM32F10X-中文参考手册》==中对应外设的寄存器说明。
在GPIO 外设中，控制端口高低控制寄存器CRH 和CRL 可以配置每个GPIO 的工作模式和工作的速度，每4 个位控制一个IO，CRH 控制端口的高八位，CRL 控制端口的低8 位，具体的看CRH 和CRL 的寄存器描述。

![5](https://github.com/Leon199601/MCU/blob/main/pic/w-5.jpg)

![6](https://github.com/Leon199601/MCU/blob/main/pic/w-6.jpg)

### 实验：使用寄存器点亮LED灯

先了解原理，打开“8-使用寄存器点亮LED灯”文件---打开“.uvprojx”，可看见三个文件，startup_stm32f10x_hd.s 、stm32f10x.h 以及main.c

#### 1、硬件连接

目标是把GPIO的引脚设置为推挽输出模式并且默认下拉，输出低电平，这样LED灯亮起了。

![7](https://github.com/Leon199601/MCU/blob/main/pic/w-7.jpg)

#### 2、启动文件

startup_stm32f10x_hd.s 文件里使用的是汇编语言，，当STM32 芯片上电启动的时候，首先会执行这里的汇编程序，从而建立起C 语言的运行环境，所以我们把这个文件称为启动文件。该文件使用的汇编指令是Cortex-M3 内核支持的指令，可参考==《Cortex-M3 权威指南》==中指令集章节。
startup_stm32f10x_hd.s 文件由官方提供，一般有需要也是在官方的基础上修改，不会自己完全重写。该文件从 ST 固件库里面找到，找到该文件后把启动文件添加到工程里面即可。不同型号的芯片以及不同编译环境下使用的汇编文件是不一样的，但功能相同。

总结启动文件的功能，如下：

初始化堆栈指针SP

初始化程序计数器指针PC

设置堆、栈的大小

初始化中断向量表

配置外部SRAM作为数据存储器（这个由用户配置，一般的开发板可没有外部SRAM）

调用SystemIni()函数配置STM32的系统时钟

设置C库的分支入口“__main”(最终用来调用main函数)

主要理解最后两点，在启动文件中有一段复位后立即执行的程序，见下代码。在实际工程中阅读时，可使用编辑器的搜索(Ctrl+F)功能查找这段代码在文件中的位置，搜索Reset_Handler 即可找到。

```assembly
;复位后执行的程序
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  __main
                IMPORT  SystemInit
                LDR     R0, =SystemInit
                BLX     R0               
                LDR     R0, =__main
                BX      R0
                ENDP
```

第一二行是程序注释，汇编里注释“；”，类似C语言“//”注释符

第三行定义子程序：Reset_Handler，PROC是子程序定义伪指令。相当于C语言中定义一个函数，函数名为Reset_Handler。

第四行EXPORT表示Reset_Handler这个子程序可供其他模块调用。相当于C语言函数声明。关键字[WEAK]表示弱定义，如果编译器发现在别处定义了同名的函数，则在链接时用别处的地址进行链接，如果其它地方没有定义，编译器也不报错，以此处地址进行链接。

第五六行IMPORT说明 SystemInit 和__main 这两个标号在其他文件，在链接的时候需要到其他文件去寻找。相当于C 语言中，从其它文件引入函数声明。以便下面对外部函数进行调用。

SystemInit 需要由我们自己实现，即我们要编写一个具有该名称的函数，用来初始化STM32 芯片的时钟，一般包括初始化AHB、APB 等各总线的时钟，需要经过一系列的配置STM32 才能达到稳定运行的状态。其实这个函数在固件库里面有提供，官方已经为我们写好。

__main 其实不是我们定义的(不要与C 语言中的main 函数混淆)，这是一个C 库函数，当编译器编译时，只要遇到这个标号就会定义这个函数，该函数的主要功能是：负责初始化栈、堆，配置系统环境，并在函数的最后调用用户编写的 main 函数，从此来到 C 的世界。

第七行把SystemInit 的地址加载到寄存器 R0。

第八行程序跳转到 R0 中的地址执行程序，即执行SystemInit 函数的内容。

第九行把__main的地址加载到寄存器R0。

第十行行程序跳转到 R0 中的地址执行程序，即执行__main 函数，执行完毕之后就去到我们熟知的 C 世界，进入main 函数。
第十行表示子程序的结束。
总之，看完这段代码后，了解到如下内容即可：

1.我们需要在外部定义一个SystemInit函数设置STM32 的时钟；

2.STM32 上电后，会执行SystemInit 函数，最后执行我们C 语言中的main 函数。

#### 3、stm32f10x.h文件

连接LED灯的GPIO引脚，通过读写寄存器来控制，寄存器映射就是给一个已经分配好地址的特殊的内存空间取的一个别名，这个特殊的内存空间就是寄存器，它可以通过指针来操作。在编程之前我们要先实现寄存器映射，寄存器映射的代码都统一写在stm32f10x.h 文件中

```c
/*片上外设基地址  */
#define PERIPH_BASE           ((unsigned int)0x40000000)

/*APB2 总线基地址 */
#define APB2PERIPH_BASE       (PERIPH_BASE + 0x10000)
/* AHB总线基地址 */
#define AHBPERIPH_BASE        (PERIPH_BASE + 0x20000)

/*GPIOC外设基地址*/
#define GPIOC_BASE            (APB2PERIPH_BASE + 0x1000)

/* GPIOC寄存器地址,强制转换成指针 */
#define GPIOC_CRL			*(unsigned int*)(GPIOC_BASE+0x00)
#define GPIOC_CRH			*(unsigned int*)(GPIOC_BASE+0x04)
#define GPIOC_IDR			*(unsigned int*)(GPIOC_BASE+0x08)
#define GPIOC_ODR			*(unsigned int*)(GPIOC_BASE+0x0C)
#define GPIOC_BSRR	  *(unsigned int*)(GPIOC_BASE+0x10)
#define GPIOC_BRR			*(unsigned int*)(GPIOC_BASE+0x14)
#define GPIOC_LCKR		*(unsigned int*)(GPIOC_BASE+0x18)

/*RCC外设基地址*/
#define RCC_BASE      (AHBPERIPH_BASE + 0x1000)
/*RCC的AHB1时钟使能寄存器地址,强制转换成指针*/
#define RCC_APB2ENR		 *(unsigned int*)(RCC_BASE+0x18)
```

最后两段是RCC 外设寄存器的地址定义，RCC 外设是用来设置时钟的，以后我们会详细分析，本实验中只要了解到使用GPIO 外设必须开启它的时钟即可。

#### 4、main文件

先编写main函数

```c
int main (void)
{
}
```

此时直接编译报错：

“Error: L6218E: Undefined symbol SystemInit (referred from startup_stm32f10x.o)”

错误提示SystemInit 没有定义。

从分析启动文件时我们知道，Reset_Handler 调用了该函数用来初始化SMT32 系统时钟，为了简单起见，我们在 main 文件里面定义一个SystemInit 空函数，什么也不做，为的是骗过编译器，把这个错误去掉。关于配置系统时钟我们在后面再写。当我们不配置系统时钟时，STM32 会把HSI 当作系统时钟，HSI=8M，由芯片内部的振荡器提供。我们在main 中添加如下函数：

```c
// 函数为空，目的是为了骗过编译器不报错
void SystemInit(void)
{	
}
```

或者还有一种方法（不建议使用此方法）：在启动文件中把有关SystemInit 的代码注释掉

```assembly
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  __main
                
                ;IMPORT  SystemInit
                ;LDR     R0, =SystemInit
                ;BLX     R0
                
                LDR     R0, =__main
                BX      R0
                ENDP
```

开启点灯之旅，在main函数中添加代码

##### 1.GPIO模式

设置GPIO引脚配置为输出模式，即配置GPIO的端口配置低寄存器CRL,CRL包含0-7号引脚，每个引脚占用4个寄存器。MODE 位用来配置输出的速度，CNF 位用来配置各种输入输出模式。在这里我们把GPIO 引脚配置为通用推挽输出，输出的速度为10M

```c
//清空控制PC2的端口位
GPIOC_CRL &= ~( 0x0F<< (4*2));	
// 配置PC2为通用推挽输出，速度为10M
GPIOC_CRL |= (1<<4*2);
```

![8](https://github.com/Leon199601/MCU/blob/main/pic/w-8.jpg)

在代码中，我们先把控制PC2 的端口位清0，然后再向它赋值“0001 b”，从而使GPIOC2 引脚设置成输出模式，速度为10M。
代码中使用了“&= ~ ”、“|=”这种操作方法是为了避免影响到寄存器中的其它位。因为寄存器不能按位读写，假如我们直接给CRL 寄存器赋值： GPIOC_CRL = 0x0000 0100;这时CRL 的的低8 ~ 11 位被设置成“0001”输出模式，但其它GPIO 引脚就有意见了，因为其它引脚的MODER 位都已被设置成输入模式(即全部被清0)。所以，为了在配置寄存器某个位的时候不影响其它位，推荐使用 “ &=~ ”和“ |= ”这两种操作方式，其中“ &=~ ”用于清0，“ |= ”用于置1

##### 2.控制引脚输出电平

输出模式，对端口位设置/清除寄存器BSRR 寄存器、端口位清除寄存器BRR 和ODR 寄存器写入参数即可控制引脚的电平状态，其中操作BSRR 和BRR 最终影响的都是ODR 寄存器，然后再通过ODR 寄存器的输出来控制GPIO。为了一步到位，我们在这里直接操作ODR 寄存器来控制GPIO 的电平

```c
// PC2 输出 低电平
GPIOC_ODR &= ~(1<<2);
```

![9](https://github.com/Leon199601/MCU/blob/main/pic/w-9.jpg)

##### 3.开启外设时钟

STM32 的 外设很多，为了降低功耗，每个外设都对应着一个时钟，在芯片刚上电的时候这些时钟都是被关闭的，如果想要外设工作，必须把相应的时钟打开。
STM32 的所有外设的时钟由一个专门的外设来管理，叫 RCC（reset and clockcontrol），RCC 在==《 STM32 中文参考手册》==的第六章。关于RCC 外设中的时钟部分，我们在后面的章节《RCC—使用HSE/HIS 配置》中有详细的讲解，这里我们暂时先了解下。所有的 GPIO 都挂载到 APB2 总线上，具体的时钟由APB2 外设时钟使能寄存器(RCC_APB2ENR)来控制

```c
// 开启GPIOC 端口时钟
RCC_APB2ENR |= (1<<4);
```

![10](https://github.com/Leon199601/MCU/blob/main/pic/w-10.jpg)

##### 4.成功点亮，水到渠成

开启时钟，配置引脚模式，控制电平，控制一个LED灯。

```c
int main(void)
{	
	// 开启GPIOC 端口时钟
	RCC_APB2ENR |= (1<<4);

	//清空控制PC2的端口位
	GPIOC_CRL &= ~( 0x0F<< (4*2));	
	// 配置PC2为通用推挽输出，速度为10M
	GPIOC_CRL |= (1<<4*2);

	// PC2 输出 低电平
	GPIOC_ODR &= ~(1<<2);
	
	while(1);
}
```

## 构建库函数雏形

为了更好的维护，要学习STM32的固件库，在固件库的基础上了解底层，学习寄存器。

### STM32函数库

固件库就是指“STM32标准函数库”，由ST公司针对STM32提供的函数接口（API），开发者调用这些函数接口来配置STM32的寄存器。开发快速、易阅读、维护成本低

![11](https://github.com/Leon199601/MCU/blob/main/pic/w-11.jpg)

### 用库来开发及学习

采用直接配置寄存器的方式开发的程序员，会列举以下原因：
(1) 具体参数更直观
(2) 程序运行占用资源少

相对于库开发的方式，直接配置寄存器方式生成的代码量的确会少一点，但因为STM32 有充足的资源，权衡库的优势与不足，绝大部分时候，我们愿意牺牲一点CPU 资源，选择库开发。一般只有在对代码运行时间要求极苛刻的地方，才用直接配置寄存器的方式代替，如频繁调用的中断服务函数。

对于库开发与直接配置寄存器的方式，就好比编程是用汇编好还是用 C 好一样。在STM32F1 系列刚推出函数库时引起程序员的激烈争论，但是，随着ST 库的完善与大家对库的了解，更多的程序员选择了库开发。现在STM32F1 系列和STM32F4 系列各有一套自己的函数库，但是它们大部分是兼容的，F1 和F4 之间的程序移植，只需要小修改即可。而如果要移植用寄存器写的程序，那简直跟脱胎换骨差不多。

用库来进行开发，市场已有定论，用户群说明了一切，但对于STM32 的学习仍然有人认为用寄存器好，而且汇编不是还没退出大学教材么？认为这种方法直观，能够了解到是配置了哪些寄存器，怎样配置寄存器。事实上，库函数的底层实现恰恰是直接配置寄存器方式的最佳例子，它代替我们完成了寄存器配置的工作，而想深入了解芯片是如何工作的话，只要直接查看库函数的最底层实现就能理解，相信你会为它严谨、优美的实现方式而陶醉，要==想修炼C 语言，就从ST 的库开始==吧。所以在以后的章节中，使用软件库是我们的重点，而且我们通过讲解库API 去效地学习STM32 的寄存器，并不至于因为用库学习，就不会用寄存器控制STM32 芯片。

### 实验：构建库函数雏形

在寄存器点亮LED的代码上完善，把代码一层层封装，实现库的最初的雏形，打开配套例程“9-自己写库—构建库函数雏形”来理解阅读

#### 1、外设寄存器结构体定义

外设寄存器地址都是基于外设基地址的偏移地址，都是在外设基地址上逐渐连续递增，每个寄存器占32个字节，和结构体里的成员类似。结构体的首地址等于外设的基地址，结构体的成员等于寄存器，成员排列顺序跟寄存器顺序一样。

在工程中“stm32f10x.h”文件中，用结构体封装GPIO及RCC外设的寄存器

```c
//寄存器的值常常是芯片外设自动更改的，即使CPU没有执行程序，也有可能发生变化
//编译器有可能会对没有执行程序的变量进行优化，所以用volatile来修饰寄存器变量

//volatile表示易变的变量，防止编译器优化，
#define		__IO	volatile	
typedef unsigned int      uint32_t;
typedef unsigned short    uint16_t;

//GPIO 寄存器结构体定义
typedef struct
{
	uint32_t CRL;	//端口配置低寄存器，		地址偏移0X00
	uint32_t CRH;	//高寄存器				0X04
	uint32_t IDR;	//端口数据输入寄存器		 0X08	
	uint32_t ODR;	//输出寄存器			   0X0C
	uint32_t BSRR;	//端口位设置/清除寄存器	0X10
	uint32_t BRR;	//端口位清除寄存器		 0X14
	uint32_t LCKR;	//端口配置锁定寄存器		0X18
}GPIO_TypeDef;
```

这段代码在每个结构体成员前增加了一个“__IO”前缀，它的原型在这段代码的第一行，代表了C 语言中的关键字“==volatile==”，在C 语言中该关键字用于表示变量是易变的，要求编译器不要优化。这些结构体内的成员，都代表着寄存器，而寄存器很多时候是由外设或STM32 芯片状态修改的，也就是说即使CPU 不执行代码修改这些变量，变量的值也有可能被外设修改、更新，所以每次使用这些变量的时候，我们都要求CPU 去该变量的地址重新访问。若没有这个关键字修饰，在某些情况下，编译器认为没有代码修改该变量，就直接从CPU 的某个缓存获取该变量值，这时可以加快执行速度，但该缓存中的是陈旧数据，与我们要求的寄存器最新状态可能会有出入。

#### 2、外设存储器映射

外设的地址定义成一个个宏，把寄存器地址和结构体地址对应起来，实现外设存储器的映射

```c
//片上外设基地址
#define  PERIPH_BASE               ((unsigned int)0x40000000)

//APB2 总线基地址
#define  APB2PERIPH_BASE          (PERIPH_BASE + 0x10000)
//AHB 总线基地址
#define  AHBPERIPH_BASE           (PERIPH_BASE + 0x20000)

//GPIO外设基地址
#define GPIOA_BASE			(APB2PERIPH_BASE + 0x0800)
#define GPIOB_BASE			(APB2PERIPH_BASE + 0x0800)
#define GPIOC_BASE			(APB2PERIPH_BASE + 0x0800)
#define GPIOD_BASE			(APB2PERIPH_BASE + 0x0800)
#define GPIOE_BASE			(APB2PERIPH_BASE + 0x0800)
#define GPIOF_BASE			(APB2PERIPH_BASE + 0x0800)
#define GPIOG_BASE			(APB2PERIPH_BASE + 0x0800)

//RCC外设基地址
#define  RCC_BASE                (AHBPERIPH_BASE + 0x1000)
```

#### 3、外设声明

```c
//GPIO 外设声明
#define GPIOA   ((GPIO_TypeDef*)GPIOA_BASE)
#define GPIOB   ((GPIO_TypeDef*)GPIOB_BASE)
#define GPIOC   ((GPIO_TypeDef*)GPIOC_BASE)
#define GPIOD   ((GPIO_TypeDef*)GPIOD_BASE)
#define GPIOE   ((GPIO_TypeDef*)GPIOE_BASE)
#define GPIOF   ((GPIO_TypeDef*)GPIOF_BASE)
#define GPIOG   ((GPIO_TypeDef*)GPIOG_BASE)

//RCC 外设声明
#define RCC     ((RCC_TypeDef*)RCC_BASE)

//RCC的AHB1时钟使能寄存器地址，强制转换成指针
#define RCC_APB2ENR     *(unsigned int *)(RCC_BASE+0x18)
```

通过强制类型转换把外设的基地址转换成GPIO_TypeDef 类型的结构体指针，然后通过宏定义把GPIOA、GPIOB 等定义成外设的结构体指针，通过外设的结构体指针就可以达到访问外设的寄存器的目的。

```c
//使用寄存器结构体指针点亮LED
int main (void)
{	
#if 0	//直接通过操作内存来控制寄存器	
	// 打开 GPIOC 端口的时钟
	RCC_APB2ENR  |=  ( (1) << 4 );
	
	// 配置IO口为输出
	GPIOC_CRL &=  ~( (0x0f) << (4*2) );
	GPIOC_CRL |=  ( (1) << (4*2) );
	
	// 控制 ODR 寄存器
	GPIOC_ODR &= ~(1<<2);
	//GPIOC_ODR |= (1<<2);

#elif 1		//通过寄存器结构体指针来控制寄存器
		// 打开 GPIOC 端口的时钟
	RCC_APB2ENR  |=  ( (1) << 4 );
	
	// 配置IO口为输出
	GPIOC->CRL &=  ~( (0x0f) << (4*2) );
	GPIOC->CRL |=  ( (1) << (4*2) );
	
	// 控制 ODR 寄存器
	GPIOC->ODR &= ~(1<<2);
	//GPIOC->ODR |= (1<<2);
#endif
}
```

除了把“_”换成了“->”，其他都跟使用寄存器点亮LED 那部分代码一样。这是因为我们现在只是实现了库函数的基础，还没有定义库函数。打好了地基，下面我们就来建高楼。接下来使用函数来封装GPIO 的基本操作，方便以后应用的时候不需要再查询寄存器，而是直接通过调用这里定义的函数来实现。我们把针对GPIO 外设操作的函数及其宏定义分别存放在“ stm32f10x_gpio.c ” 和“stm32f10x_gpio.h”文件中，这两个文件需要自己新建。

#### 4、定义位操作函数

“stm32f10x_gpio.c”中控制引脚输出高低电平

```c
/*
 *函数功能：设置引脚为高电平
 *参数说明：GPIOx：该参数GPIO_TypeDef类型的指针，指向GPIO端口地址
 *GPIO_Pin:选择要设置的GPIO端口引脚，可输入宏GPIO_Pin_0-15,
 *		   表示GPIOx端口的0-15号引脚。
 */
void GPIO_SetBits(GPIO_TypeDef *GPIOx,uint16_t GPIO_Pin)
{
    /*设置GPIOx端口BSRR寄存器的第GPIO_Pin位，使其输出高电平*/
    /*因为BSRR寄存器写0不影响，
      宏GPIO_Pin只是对应位为1，其他位均为0，所有可以直接赋值*/
    
	GPIOx->BSRR |= GPIO_Pin;
}

/*
 *函数功能：设置引脚为低电平
 *参数说明：GPIOx：该参数GPIO_TypeDef类型的指针，指向GPIO端口地址
 *GPIO_Pin:选择要设置的GPIO端口引脚，可输入宏GPIO_Pin_0-15,
 *		   表示GPIOx端口的0-15号引脚。
 */
void GPIO_ResetBits( GPIO_TypeDef *GPIOx,uint16_t GPIO_Pin )
{
    /*设置GPIOx端口BRR寄存器的第GPIO_Pin位，使其输出高电平*/
    /*因为BRR寄存器写0不影响，
      宏GPIO_Pin只是对应位为1，其他位均为0，所有可以直接赋值*/
    
	GPIOx->BRR |= GPIO_Pin;
}
```

![12](https://github.com/Leon199601/MCU/blob/main/pic/w-12.jpg)

![13](https://github.com/Leon199601/MCU/blob/main/pic/w-13.jpg)

控制各种端口引脚的范例代码

```c
//控制GPIO的引脚10输出高低电平
GPIO_SetBits(GPIOB,(uint16_t)(1<<10));
GPIO_ResetBits(GPIOB,(uint16_t)(1<<10));
```

设置引脚好时，不方便，为此把表示16个引脚的操作数都定义成宏

```c
/*GPIO引脚号定义*/
#define GPIO_Pin_0    ((uint16_t)0x0001)  /*!< 选择Pin0 */    //(00000000 00000001)b
#define GPIO_Pin_1    ((uint16_t)0x0002)  /*!< 选择Pin1 */    //(00000000 00000010)b
#define GPIO_Pin_2    ((uint16_t)0x0004)  /*!< 选择Pin2 */    //(00000000 00000100)b
#define GPIO_Pin_3    ((uint16_t)0x0008)  /*!< 选择Pin3 */    //(00000000 00001000)b
#define GPIO_Pin_4    ((uint16_t)0x0010)  /*!< 选择Pin4 */    //(00000000 00010000)b
#define GPIO_Pin_5    ((uint16_t)0x0020)  /*!< 选择Pin5 */    //(00000000 00100000)b
#define GPIO_Pin_6    ((uint16_t)0x0040)  /*!< 选择Pin6 */    //(00000000 01000000)b
#define GPIO_Pin_7    ((uint16_t)0x0080)  /*!< 选择Pin7 */    //(00000000 10000000)b

#define GPIO_Pin_8    ((uint16_t)0x0100)  /*!< 选择Pin8 */    //(00000001 00000000)b
#define GPIO_Pin_9    ((uint16_t)0x0200)  /*!< 选择Pin9 */    //(00000010 00000000)b
#define GPIO_Pin_10   ((uint16_t)0x0400)  /*!< 选择Pin10 */   //(00000100 00000000)b
#define GPIO_Pin_11   ((uint16_t)0x0800)  /*!< 选择Pin11 */   //(00001000 00000000)b
#define GPIO_Pin_12   ((uint16_t)0x1000)  /*!< 选择Pin12 */   //(00010000 00000000)b
#define GPIO_Pin_13   ((uint16_t)0x2000)  /*!< 选择Pin13 */   //(00100000 00000000)b
#define GPIO_Pin_14   ((uint16_t)0x4000)  /*!< 选择Pin14 */   //(01000000 00000000)b
#define GPIO_Pin_15   ((uint16_t)0x8000)  /*!< 选择Pin15 */   //(10000000 00000000)b
#define GPIO_Pin_All  ((uint16_t)0xFFFF)  /*!< 选择全部引脚*/ //(11111111 11111111)b
```

控制设置整个端口0-15所有引脚

```c
//控制GPIO的引脚10输出高低电平
GPIO_SetBits(GPIOB,GPIO_Pin_10);
GPIO_ResetBits(GPIOB,GPIO_Pin_10);
```

使用以上代码控制GPIO，我们就不需要再看寄存器了，直接从函数名和输入参数就可以直观看出这个语句要实现什么操作。(英文中―Set‖表示“置位”，即高电平，“Reset”表示“复位”，即低电平)

#### 5、定义初始化结构体GPIO_InitTypeDef

定义位操作函数后，控制GPIO 输出电平的代码得到了简化，但在控制GPIO 输出电平前还需要初始化GPIO 引脚的各种模式，这部分代码涉及的寄存器有很多，我们希望初始化GPIO 也能以如此简单的方法去实现。为此，我们先根据GPIO 初始化时涉及到的初始化参数以结构体的形式封装起来，声明一个名为GPIO_InitTypeDef 的结构体类型

```c
typedef struct
{
	uint16_t GPIO_Pin;		/*选择要配置的GPIO引脚*/
	uint16_t GPIO_Speed;	/*选择GPIO引脚的速率*/
	uint16_t GPIO_Mode;		/*选择GPIO引脚的工作模式*/
}GPIO_InitTypeDef;
```

该函数能根据这个变量值中的内容去配置寄存器，从而实现GPIO 的初始化。

#### 6、定义引脚模式的枚举类型

上面定义的结构体很直接，美中不足的是在对结构体中各个成员赋值实现某个功能时还需要查询手册的寄存器说明，我们不希望每次用到的时候都要去查询手册，我们可以使用C 语言中的枚举定义功能，根据手册把每个成员的所有取值都定义好，具体代码

```c
typedef enum
{ 
  GPIO_Speed_10MHz = 1,         // 10MHZ        (01)b
  GPIO_Speed_2MHz,              // 2MHZ         (10)b
  GPIO_Speed_50MHz              // 50MHZ        (11)b
}GPIOSpeed_TypeDef;

typedef enum
{ GPIO_Mode_AIN = 0x0,           // 模拟输入     (0000 0000)b
  GPIO_Mode_IN_FLOATING = 0x04,  // 浮空输入     (0000 0100)b
  GPIO_Mode_IPD = 0x28,          // 下拉输入     (0010 1000)b
  GPIO_Mode_IPU = 0x48,          // 上拉输入     (0100 1000)b
  
  GPIO_Mode_Out_OD = 0x14,       // 开漏输出     (0001 0100)b
  GPIO_Mode_Out_PP = 0x10,       // 推挽输出     (0001 0000)b
  GPIO_Mode_AF_OD = 0x1C,        // 复用开漏输出 (0001 1100)b
  GPIO_Mode_AF_PP = 0x18         // 复用推挽输出 (0001 1000)b
}GPIOMode_TypeDef;
```

关于速度的枚举类型很好理解。

关于模式的枚举类型，通过表格来梳理

![14](https://github.com/Leon199601/MCU/blob/main/pic/w-14.jpg)

转化成二进制之后，就比较容易发现规律。bit4 用来区分端口是输入还是输出，0 表示输入，1 表示输出，bit2 和bit3 对应寄存器的CNFY[1:0]位，是我们真正要写入到CRL 和CRH 这两个端口控制寄存器中的值。bit0 和bit1 对应寄存器的MODEY[1:0]位，这里我们暂不初始化，在GPIO_Init()初始化函数中用来跟GPIOSpeed 的值相加即可实现速率的配置。有关具体的代码分析见GPIO_Init()库函数。其中在下拉输入和上拉输入中我们设置bit5 和bit6 的值为01 和10 来以示区别。

有了这些枚举定义，我们的GPIO_InitTypeDef 结构体就可以使用枚举类型来限定输入参数，具体见代码

```c
typedef struct
{
	uint16_t GPIO_Pin;
	GPIOSpeed_TypeDef GPIO_Speed;
	GPIOMode_TypeDef GPIO_Mode;
}GPIO_InitTypeDef;
```

如果不使用枚举类型，仍使用“uint16_t”类型来定义结构体成员，那么成员值的范围就是0-255，而实际上这些成员都只能输入几个数值。所以使用枚举类型可以对结构体成员起到限定输入的作用，只能输入相应已定义的枚举值。
利用这些枚举定义，给GPIO_InitTypeDef 结构体类型赋值配置就变得非常直观，

```c
GPIO_InitTypeDef  GPIO_InitStructure;

GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
```

#### 7、定义GPIO初始化函数

对初始化结构体赋值后，把它输入到GPIO 初始化函数，由它来实现寄存器配置。

```c
/*
 *函数功能：初始化引脚模式
 *参数说明：GPIOx，该参数为GPIO_TypeDef类型的指针，指向GPIO端口的地址
 *		  GPIO_InitTypeDef：GPIO_InitTypeDef结构体指针，指向初始化变量
 */
void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct)
{
  uint32_t currentmode = 0x00, currentpin = 0x00, pinpos = 0x00, pos = 0x00;
  uint32_t tmpreg = 0x00, pinmask = 0x00;
  
/*---------------------- GPIO 模式配置 --------------------------*/
  // 把输入参数GPIO_Mode的低四位暂存在currentmode
  currentmode = ((uint32_t)GPIO_InitStruct->GPIO_Mode) & ((uint32_t)0x0F);
	
  // bit4是1表示输出，bit4是0则是输入 
  // 判断bit4是1还是0，即首选判断是输入还是输出模式
  if ((((uint32_t)GPIO_InitStruct->GPIO_Mode) & ((uint32_t)0x10)) != 0x00)
  { 
	// 输出模式则要设置输出速度
    currentmode |= (uint32_t)GPIO_InitStruct->GPIO_Speed;
  }
/*-------------GPIO CRL 寄存器配置 CRL寄存器控制着低8位IO- -------*/
  // 配置端口低8位，即Pin0~Pin7
  if (((uint32_t)GPIO_InitStruct->GPIO_Pin & ((uint32_t)0x00FF)) != 0x00)
  {
	// 先备份CRL寄存器的值
    tmpreg = GPIOx->CRL;
		
	// 循环，从Pin0开始配对，找出具体的Pin
    for (pinpos = 0x00; pinpos < 0x08; pinpos++)
    {
	 // pos的值为1左移pinpos位
      pos = ((uint32_t)0x01) << pinpos;
      
	  // 令pos与输入参数GPIO_PIN作位与运算，为下面的判断作准备
      currentpin = (GPIO_InitStruct->GPIO_Pin) & pos;
			
	  //若currentpin=pos,则找到使用的引脚
      if (currentpin == pos)
      {
		// pinpos的值左移两位（乘以4），因为寄存器中4个寄存器位配置一个引脚
        pos = pinpos << 2;
       //把控制这个引脚的4个寄存器位清零，其它寄存器位不变
        pinmask = ((uint32_t)0x0F) << pos;
        tmpreg &= ~pinmask;
				
        // 向寄存器写入将要配置的引脚的模式
        tmpreg |= (currentmode << pos);  
				
		// 判断是否为下拉输入模式
        if (GPIO_InitStruct->GPIO_Mode == GPIO_Mode_IPD)
        {
		  // 下拉输入模式，引脚默认置0，对BRR寄存器写1可对引脚置0
          GPIOx->BRR = (((uint32_t)0x01) << pinpos);
        }				
        else
        {
          // 判断是否为上拉输入模式
          if (GPIO_InitStruct->GPIO_Mode == GPIO_Mode_IPU)
          {
		    // 上拉输入模式，引脚默认值为1，对BSRR寄存器写1可对引脚置1
            GPIOx->BSRR = (((uint32_t)0x01) << pinpos);
          }
        }
      }
    }
		// 把前面处理后的暂存值写入到CRL寄存器之中
    GPIOx->CRL = tmpreg;
  }
/*-------------GPIO CRH 寄存器配置 CRH寄存器控制着高8位IO- -----------*/
  // 配置端口高8位，即Pin8~Pin15
  if (GPIO_InitStruct->GPIO_Pin > 0x00FF)
  {
		// // 先备份CRH寄存器的值
    tmpreg = GPIOx->CRH;
		
	// 循环，从Pin8开始配对，找出具体的Pin
    for (pinpos = 0x00; pinpos < 0x08; pinpos++)
    {
      pos = (((uint32_t)0x01) << (pinpos + 0x08));
			
      // pos与输入参数GPIO_PIN作位与运算
      currentpin = ((GPIO_InitStruct->GPIO_Pin) & pos);
			
	 //若currentpin=pos,则找到使用的引脚
      if (currentpin == pos)
      {
		//pinpos的值左移两位（乘以4），因为寄存器中4个寄存器位配置一个引脚
        pos = pinpos << 2;
        
	    //把控制这个引脚的4个寄存器位清零，其它寄存器位不变
        pinmask = ((uint32_t)0x0F) << pos;
        tmpreg &= ~pinmask;
				
        // 向寄存器写入将要配置的引脚的模式
        tmpreg |= (currentmode << pos);
        
		// 判断是否为下拉输入模式
        if (GPIO_InitStruct->GPIO_Mode == GPIO_Mode_IPD)
        {
		  // 下拉输入模式，引脚默认置0，对BRR寄存器写1可对引脚置0
          GPIOx->BRR = (((uint32_t)0x01) << (pinpos + 0x08));
        }
         // 判断是否为上拉输入模式
        if (GPIO_InitStruct->GPIO_Mode == GPIO_Mode_IPU)
        {
		  // 上拉输入模式，引脚默认值为1，对BSRR寄存器写1可对引脚置1
          GPIOx->BSRR = (((uint32_t)0x01) << (pinpos + 0x08));
        }
      }
    }
	// 把前面处理后的暂存值写入到CRH寄存器之中
    GPIOx->CRH = tmpreg;
  }
}
```

这个函数有GPIOx 和GPIO_InitStruct 两个输入参数，分别是GPIO 外设指针和GPIO初始化结构体指针。分别用来指定要初始化的GPIO 端口及引脚的工作模式。

要深入了解该函数，得配合GPIO引脚工作模式真值表

1）先取得GPIO_Mode 的值，判断bit4 是1 还是0 来判断是输出还是输入。如果是输出则设置输出速率，即加上GPIO_Speed 的值，输入没有速率之说，不用设置。
2） 配置CRL 寄存器。通过GPIO_Pin 的值计算出具体需要初始化哪个引脚，算出后，然后把需要配置的值写入到CRL 寄存器中，具体分析见代码注释。这里有一个比较有趣的是上/下拉输入并不是直接通过配置某一个寄存器来实现的，而是通过写BSRR 或者BRR 寄存器来实现。这让很多只看手册没看固件库底层源码的人摸不着头脑，因为手册的寄存器说明中没有明确的指出如何配置上拉/下拉，具体见图
3） 配置CRH 寄存器过程同CRL。

![15](https://github.com/Leon199601/MCU/blob/main/pic/w-15.jpg)

#### 8、使用函数点亮LED灯

```c
//使用固件库点亮LED
int main(void)
{
	GPIO_InitTypeDef  GPIO_InitStructure;
	
	// 开启GPIO端口的时钟
	RCC_APB2ENR |= (1<<4);
	
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOC, &GPIO_InitStructure);
    GPIO_ResetBits(GPIOC,GPIO_Pin_2);

  while(1)
  {
		GPIO_SetBits(GPIOC,GPIO_Pin_2);
		Delay(0xFFFF);		//延时函数
		GPIO_ResetBits(GPIOC,GPIO_Pin_2);
		Delay(0xFFFF);
	}
}
```

使用函数来控制LED 灯与之前直接控制寄存器已经有了很大的区别：main 函数中先定义了一个GPIO 初始化结构体变量GPIO_InitStructure，然后对该变量的各个成员按点亮LED 灯所需要的GPIO 配置模式进行赋值，赋值后，调用GPIO_Init 函数，让它根据结构体成员值对GPIO 寄存器写入控制参数，完成GPIO 引脚初始化。控制电平时，直接使用GPIO_SetBits 和GPIO_Resetbits 函数控制输出。如若对其它引脚进行不同模式的初始化，只要修改GPIO 初始化结构体GPIO_InitStructure 的成员值，把新的参数值输入到GPIO_Init 函数再调用即可。

#### 9、下载验证

#### 10、总结

与直接配置寄存器相比，执行效率上看会耗时间：初始化变量赋值的过程、库函数被调用的时候耗费调用时间，函数内部，对输入参数转换所需要的额外运算也消耗时间（如GPIO运算求出引脚号时），而宏、枚举灯解释操作是作编译过程完成的，不消耗内核的时间。

库函数的优点是可以快速上手STM32控制器，交流方便，查错简单。

## 初识STM32标准库

ST公司提供的标准软件库，包含了STM32芯片所有寄存器的控制操作，方便我们控制STM32芯片。

### CMSIS标准及库层次关系

因为基于Cortex 系列芯片采用的内核都是相同的，区别主要为核外的片上外设的差异，这些差异却导致软件在同内核，不同外设的芯片上移植困难。为了解决不同的芯片厂商生产的Cortex 微控制器软件的兼容性问题，ARM 与芯片厂商建立了CMSIS 标准。

所谓CMSIS 标准，实际是新建了一个软件抽象层。

![16](https://github.com/Leon199601/MCU/blob/main/pic/w-16.jpg)

CMSIS 标准中最主要的为CMSIS 核心层，它包括了：
 内核函数层：其中包含用于访问内核寄存器的名称、地址定义，主要由ARM 公司提供。
 设备外设访问层：提供了片上的核外外设的地址和中断定义，主要由芯片生产商提供。

可见CMSIS 层位于硬件层与操作系统或用户层之间，提供了与芯片生产商无关的硬件抽象层，可以为接口外设、实时操作系统提供简单的处理器软件接口，屏蔽了硬件差异，这对软件的移植是有极大的好处的。STM32 的库，就是按照CMSIS 标准建立的。

#### 库目录、文件简介

STM32标准库可从官网下载，本书采用3.5.0库文件，目录：STM32F10x_StdPeriph_Lib_V3.5.0\

![17](https://github.com/Leon199601/MCU/blob/main/pic/w-17.jpg)

 Libraries：文件夹下是驱动库的源代码及启动文件，这个非常重要，我们要使用的固件库就在这个文件夹里面。
 Project ：文件夹下是用驱动库写的例子和工程模板，其中那些为每个外设写好的例程对我们非常有用，我们在学习的时候就可以参考这里面的例程，非常全面，简直就是穷尽了外设的所有功能。
 Utilities：包含了基于ST 官方实验板的例程，不需要用到，略过即可。
 stm32f10x_stdperiph_lib_um.chm： 库帮助文档，这个很有用，不喜欢直接看源码的可以在合理查询每个外设的函数说明，非常详细。这是一个已经编译好的HTML 文件，主要讲述如何使用驱动库来编写自己的应用程序。说得形象一点，这个HTML 就是告诉我们：ST 公司已经为你写好了每个外设的驱动了，想知道如何运用这些例子就来向我求救吧。不幸的是，这个帮助文档是英文的，这对很多英文不好的朋友来说是一个很大的障碍。但这里要告诉大家，英文仅仅是一种工具，绝对不能让它成为我们学习的障碍。其实这些英文还是很简单的，我们需要的是拿下它的勇气。

在使用库开发时，我们需要把libraries 目录下的库函数文件添加到工程中，并查阅库帮助文档来了解ST 提供的库函数，这个文档说明了每一个库函数的使用方法。

Libraries文件夹下存放在CMSIS和STM32F10x_StdPeriph_Driver 文件夹

##### 1.CMSIS文件夹

![18](https://github.com/Leon199601/MCU/blob/main/pic/w-18.jpg)

黄色框为需要的内容

内核相关的文件

Core_cm3.h 头文件里面实现了内核的寄存器映射，对应外设头文件stm32f10x.h，区别就是一个针对内核的外设，一个针对片上（内核之外）的外设。core_cm3.c 文件实现了一下操作内核外设寄存器的函数，用的比较少。

core_cm3.h 头文件中包含了“stdint.h” 这个头文件，这是一个ANSI C 文件，是独立于处理器之外的，就像我们熟知的C 语言头文件 “stdio.h” 文件一样。位于RVMDK 这个软件的安装目录下，主要作用是提供一些类型定义。

```c
/*exact-width signed integer types */
typedef		signed			char int8_t;
typedef		signed short 	 int int16_t;
typedef		signed		 	 int int32_t;
typedef		signed		 __int64 int64_t;

/*exact-width unsigned integer types */
typedef		signed			char int8_t;
typedef		signed short 	 int int16_t;
typedef		signed		 	 int int32_t;
typedef		signed		 __int64 int64_t;
```

这些新类型定义屏蔽了在不同芯片平台时，出现的诸如int 的大小是16 位，还是32 位的差异。所以在我们以后的程序中，都将使用新类型如uint8_t 、uint16_t 等。
在稍旧版的程序中还经常会出现如u8、u16、u32 这样的类型，分别表示的无符号的8位、16 位、32 位整型。初学者碰到这样的旧类型感觉一头雾水，它们定义的位置在STM32f10x.h 文件中。建议在以后的新程序中尽量使用uint8_t 、uint16_t 类型的定义。

启动文件

启动文件放在startup/arm 这个文件夹下面，这里面启动文件有很多个，不同型号的单片机用的启动文件不一样，有关每个启动文件的详细说明见表

![19](https://github.com/Leon199601/MCU/blob/main/pic/w-19.jpg)

开发板STM32F103VET6 或者STM32F103ZET6 的FLASH 都是512K，属于基本型的大容量产品，启动文件统一选择startup_stm32f10x_hd.s。

stm32f10x.h

这个头文件实现了片上外设的所以寄存器的映射，是一个非常重要的头文件，在内核中与之想对应的头文件是core_cm3.h。

system_stm32f10x.c

该文件实现了STM32的时钟配置，操作的是片上RCC这个外设。系统在上电之后，首选会执行由汇编编写的启动文件，启动文件中的复位函数中调用的SystemInit 函数就在这个文件里面定义。调用完之后，系统的时钟就被初始化成72M。如果后面我们需要重新配置系统时钟，我们就可以参考这个函数重写。为了维持库的完整性，我们不会直接在这个文件里面修改时钟配置函数。

##### 2.STM32F10x_StdPeriph_Driver 文件夹

![20](https://github.com/Leon199601/MCU/blob/main/pic/w-20.jpg)

文件属于CMSIS 之外的、芯片片上外设部分。src 里面是每个设备外设的驱动源程序，inc 则是相对应的外设头文件。src 及inc 文件夹是ST 标准库的主要内容，甚至不少人直接认为ST 标准库就是指这些文件，可见其重要性。

在src 和inc 文件夹里的就是ST 公司针对每个STM32 外设而编写的库函数文件，每个外设对应一个 .c 和 .h 后缀的文件。我们把这类外设文件统称为：stm32f10x_ppp.c 或stm32f10x_ppp.h 文件，PPP 表示外设名称。如在上一章中我们自建的stm32f10x_gpio.c 及stm32f10x_gpio.h 文件，就属于这一类。

如针对模数转换(ADC)外设，在src 文件夹下有一个stm32f10x_adc.c 源文件，在inc 文件夹下有一个stm32f10x_adc.h 头文件，若我们开发的工程中用到了STM32 内部的ADC，则至少要把这两个文件包含到工程里。

这两个文件夹中，还有一个很特别的misc.c 文件，这个文件提供了外设对内核中的NVIC(中断向量控制器)的访问函数，在配置中断时，我们必须把这个文件添加到工程中。

##### 3.stm32f10x_it.c、 stm32f10x_conf.h 和system_stm32f10x.c 文件

文件目录：STM32F10x_StdPeriph_Lib_V3.5.0\Project\STM32F10x_StdPeriph_Template

建立完整的工程时，需要添加这四个文件（stm32f10x_it.h）

stm32f10x_it.c：该文件是专门用来编写中断服务函数的，已经定义了一些系统异常（特殊中断）的接口，普通中断服务函数是自己添加。

system_stm32f10x.c：该文件包含了STM32芯片上电后初始化系统时钟、扩展外部存储器用的函数，之前提到供启动文件调用的“SystemInit”函数，上电后初始化时钟，就存储在该文件中，调用该函数后，系统时钟被初始化为72MHz，如需要修改，应该在做系统时钟配置的时候另外重新写时钟配置函数，以保证库的完整性。

stm32f10x_conf.h：该文件被包含进stm32f10x.h文件，用一个头文件stm32f10x_conf.h 把这些外设的头文件都包含在里面。stm32f10x_conf.h中的代码

```c
#include "stm32f10x_adc.h"
#include "stm32f10x_bkp.h"
#include "stm32f10x_can.h"
#include "stm32f10x_cec.h"
#include "stm32f10x_crc.h"
#include "stm32f10x_dac.h"
#include "stm32f10x_dbgmcu.h"
#include "stm32f10x_dma.h"
#include "stm32f10x_exti.h"
#include "stm32f10x_flash.h"
#include "stm32f10x_fsmc.h"
#include "stm32f10x_gpio.h"
#include "stm32f10x_i2c.h"
#include "stm32f10x_iwdg.h"
#include "stm32f10x_pwr.h"
#include "stm32f10x_rcc.h"
#include "stm32f10x_rtc.h"
#include "stm32f10x_sdio.h"
#include "stm32f10x_spi.h"
#include "stm32f10x_tim.h"
#include "stm32f10x_usart.h"
#include "stm32f10x_wwdg.h"
#include "misc.h" 
```

stm32f10x_conf.h该文件还可配置是否使用“断言”编译选项

```c
#ifdef  USE_FULL_ASSERT

/**
  * @brief  The assert_param macro is used for function's parameters check.
  * @param  expr: If expr is false, it calls assert_failed function which reports 
  *         the name of the source file and the source line number of the call 
  *         that failed. If expr is true, it returns no value.
  * @retval None
  */
  #define assert_param(expr) ((expr) ? (void)0 : assert_failed((uint8_t *)__FILE__, __LINE__))
/* Exported functions ------------------------------------------------------- */
  void assert_failed(uint8_t* file, uint32_t line);
#else
  #define assert_param(expr) ((void)0)
#endif /* USE_FULL_ASSERT */
```

ST标准库的函数中，一般会包含输入参数检查，即上述代码中的“assert_param”宏，当参数不符合要求时，会调用“assert_failed”函数，这个函数默认是空的。

实际开发中使用断言时，先通过定义USE_FULL_ASSERT 宏来使能断言，然后定义“assert_failed”函数，通常我们会让它调用printf 函数输出错误说明。使能断言后，程序运行时会检查函数的输入参数，当软件经过测试，可发布时，会取消USE_FULL_ASSERT宏来去掉断言功能，使程序全速运行。

#### 库个文件间的关系

从整体上把握各个文件在库工程中的层次或关系

![21](https://github.com/Leon199601/MCU/blob/main/pic/w-21.jpg)

实际的使用库开发工程的过程中，把位于CMSIS 层的文件包含进工程，除了特殊系统时钟需要修改system_stm32f10x.c，其它文件丝毫不用修改，也不建议修改。
对于位于用户层的几个文件，就是我们在使用库的时候，针对不同的应用对库文件进行增删（用条件编译的方法增删）和改动的文件。

### 使帮助文档

官方的帮助手册是最好的教程，包含了所有开发过程中遇到的问题

##### 常用官方资料

 《STM32F10X-中文参考手册》
这个文件全方位介绍了STM32 芯片的各种片上外设，它把STM32 的时钟、存储器架构、及各种外设、寄存器都描述得清清楚楚。当我们对STM32 的外设感到困惑时，可查阅这个文档。以直接配置寄存器方式开发的话，查阅这个文档寄存器部分的频率会相当高，但这样效率太低了。
 《STM32 规格书》
本文档相当于STM32 的datasheet，包含了STM32 芯片所有的引脚功能说明及存储器架构、芯片外设架构说明。后面我们使用STM32 其它外设时，常常需要查找这个手册，了解外设对应到STM32 的哪个GPIO 引脚。
 《Cortex™-M3 内核编程手册》
本文档由ST 公司提供，主要讲解STM32 内核寄存器相关的说明，例如系统定时器、NVIC 等核外设的寄存器。这部分的内容是《STM32F10X-中文参考手册》没涉及到的内核部分的补充。相对来说，本文档虽然介绍了内核寄存器，但不如以下两个文档详细，要了解内核时，可作为以下两个手册的配合资料使用。
 《Cortex-M3 权威指南》。
这个手册是由ARM 公司提供的，它详细讲解了Cortex 内核的架构和特性，要深入了解Cortex-M 内核，这是首选，经典中的经典。这个手册也被翻译成中文，出版成书，我们配套的资料里面有提供中文版的电子版。
 《stm32f10x_stdperiph_lib_um.chm》
这个就是本章提到的库的帮助文档，在使用库函数时，我们最好通过查阅此文件来了解标准库提供了哪些外设、函数原型或库函数的调用的方法。也可以直接阅读源码里面的函数的函数说明。

##### 初识库函数

调用这些库函数，来控制STM32。必须知道调用函数的功能、可传入的参数及其意义、和函数的返回值。

会查库函数就可以，库帮助文档《stm32f10x_stdperiph_lib_um.chm》

标签目录：Modules\STM32F10x_StdPeriph_Driver\GPIO\Functions\GPIO_SetBits

![22](https://github.com/Leon199601/MCU/blob/main/pic/w-22.jpg)

利用这个文档，我们即使没有去看它的具体源代码，也知道要怎么利用它了。
如GPIO_SetBits，函数的原型为void GPIO_SetBits(GPIO_TypeDef * GPIOx , uint16_tGPIO_Pin)。它的功能是：输入一个类型为GPIO_TypeDef 的指针GPIOx 参数，选定要控制的GPIO 端口；输入GPIO_Pin_x 宏，其中x 指端口的引脚号，指定要控制的引脚。
其中输入的参数 GPIOx 为ST 标准库中定义的自定义数据类型，这两个传入参数均为结构体指针。初学时，我们并不知道如GPIO_TypeDef 这样的类型是什么意思，可以点击函数原型中带下划线的 GPIO_TypeDef 就可以查看这个类型的声明了。
就这样初步了解了一下库函数，读者就可以发现STM32 的库是写得很优美的。每个函数和数据类型都符合见名知义的原则，当然，这样的名称写起来特别长，而且对于我们来说要输入这么长的英文，很容易出错，所以在开发软件的时候，在用到库函数的地方，直接把库帮助文档中的函数名称复制粘贴到工程文件就可以了。而且，配合MDK 软件的代码自动补全功能，可以减少输入量。
有的用户觉得使用库文档麻烦，也可以直接查阅STM32 标准库的源码，库帮助文档的说明都是根据源码生成的，所以直接看源码也可以了解函数功能。

## 新建工程-库函数版

使用库建立一个空的工程，作为工程模板。

### 新建工程

#### 新建本地工程文件夹

在本地电脑新建一个“工程模板”文件夹，在它之下新建6个文件夹

![23](https://github.com/Leon199601/MCU/blob/main/pic/w-23.jpg)

| 名称      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| Doc       | 工程说明.txt                                                 |
| Libraries | CMSIS:里面放着跟CM2内核有关的库文件                          |
|           | STM32F10x_StdPeriph_Driver:STM32外设库文件                   |
| Listing   | 暂时为空                                                     |
| Outout    | 暂时为空                                                     |
| Project   | 暂时为空                                                     |
| User      | stm32f10x_conf.h:用来配置库的头文件                          |
|           | stm32f10x_it.h、stm32f10x_it.c：中断相关的函数都在这个文件编写，暂时为空 |
|           | main.c：main函数文件                                         |

#### 新建工程

打开KEIL5，新建一个工程

![24](https://github.com/Leon199601/MCU/blob/main/pic/w-24.jpg)

##### 1.选择CPU型号

MINI选STM32F103RC型号

![25](https://github.com/Leon199601/MCU/blob/main/pic/w-25.jpg)

##### 2.在线添加库文件

后期手动添加库文件

##### 3.添加组文件夹

