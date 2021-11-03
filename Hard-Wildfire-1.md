# 野火F103MINI板

## GPIO输出---使用固件库点亮LED

本章参考资料：《STM32F10X-中文参考手册》GPIO 和RCC 章节、库帮助文档《stm32f10x_stdperiph_lib_um》。

利用库建立好的工程模板，就可以方便地使用STM32 标准库编写应用程序了，可以说从这一章我们真正开始迈入STM32 固件库开发的大门。

### 硬件设计

![1](https://github.com/Leon199601/MCU/blob/main/pic/w1-1.jpg)

### 软件设计

为了使工程更加有条理，我们把LED 灯控制相关的代码独立分开存储，方便以后移植。在“工程模板”之上新建“bsp_led.c”及“bsp_led.h”文件，其中的“bsp”即BoardSupport Packet 的缩写(板级支持包)，是针对具体的MINI 这块开发板而写的驱动文件。bsp文件不属于STM32 标准库的内容，是由我们自己根据应用需要编写的。

#### 编程要点

1.使能GPIO端口时钟

2.初始化GPIO目标引脚为推挽输出模式

3.编写简单测试程序，控制GPIO引脚输出高低电平

#### 代码分析

##### 1、LED灯引脚宏定义

在编写应用程序的过程中，要考虑更改硬件环境的情况，例如LED 灯的控制引脚与当前的不一样，我们希望程序只需要做最小的修改即可在新的环境正常运行。这个时候一般把硬件相关的部分使用宏来封装，若更改了硬件环境，只修改这些硬件相关的宏即可，这些定义一般存储在头文件，即本例子中的“bsp_led.h”文件中。

```c
/* 定义LED连接的GPIO端口, 用户只需要修改下面的代码即可改变控制的LED引脚 */
#define LED1_GPIO_PORT    	GPIOC		              /* GPIO端口 */
#define LED1_GPIO_CLK 	    RCC_APB2Periph_GPIOC		/* GPIO端口时钟 */
#define LED1_GPIO_PIN			GPIO_Pin_2			        

#define LED2_GPIO_PORT    	GPIOC			              /* GPIO端口 */
#define LED2_GPIO_CLK 	    RCC_APB2Periph_GPIOC		/* GPIO端口时钟 */
#define LED2_GPIO_PIN		GPIO_Pin_3	
```

以上代码分别把控制LED 灯的GPIO 端口、GPIO 引脚号以及GPIO 端口时钟封装起来了。在实际控制的时候我们就直接用这些宏，以达到应用代码跟硬件无关的效果。
其中的GPIO 时钟宏“RCC_APB2Periph_GPIOB”是STM32 标准库定义的GPIO 端口时钟相关的宏，它的作用与“GPIO_Pin_x”这类宏类似，是用于指示寄存器位的，方便库函数使用，下面初始化GPIO 时钟的时候可以看到它的用法。

##### 2、控制LED灯亮灭状态的宏定义

为了方便控制LED 灯，我们把LED 灯常用的亮、灭及状态反转的控制也直接定义成宏

```c
/** the macro definition to trigger the led on or off 
  * 1 - off
  *0 - on
  */
#define ON  0
#define OFF 1

/* 使用标准的固件库控制IO*/
#define LED1(a)	if (a)	\
					GPIO_SetBits(LED1_GPIO_PORT,LED1_GPIO_PIN);\
					else		\
					GPIO_ResetBits(LED1_GPIO_PORT,LED1_GPIO_PIN)

#define LED2(a)	if (a)	\
					GPIO_SetBits(LED2_GPIO_PORT,LED2_GPIO_PIN);\
					else		\
					GPIO_ResetBits(LED2_GPIO_PORT,LED2_GPIO_PIN)

/* 直接操作寄存器的方法控制IO */
#define	digitalHi(p,i)		 {p->BSRR=i;}	 //输出为高电平		
#define digitalLo(p,i)		 {p->BRR=i;}	 //输出低电平
#define digitalToggle(p,i) {p->ODR ^=i;} //输出反转状态

/* 定义控制IO的宏 */
#define LED1_TOGGLE		 digitalToggle(LED1_GPIO_PORT,LED1_GPIO_PIN)
#define LED1_OFF		   digitalHi(LED1_GPIO_PORT,LED1_GPIO_PIN)
#define LED1_ON			   digitalLo(LED1_GPIO_PORT,LED1_GPIO_PIN)

#define LED2_TOGGLE		 digitalToggle(LED2_GPIO_PORT,LED2_GPIO_PIN)
#define LED2_OFF		   digitalHi(LED2_GPIO_PORT,LED2_GPIO_PIN)
#define LED2_ON			   digitalLo(LED2_GPIO_PORT,LED2_GPIO_PIN)
```

这部分宏控制LED 亮灭的操作是直接向BSRR、BRR 和ODR 这三个寄存器写入控制指令来实现的，对BSRR 写1 输出高电平，对BRR 写1 输出低电平，对ODR 寄存器某位进行异或操作可反转位的状态。
代码中的“\”是C 语言中的续行符语法，表示续行符的下一行与续行符所在的代码是同一行。代码中因为宏定义关键字“#define”只是对当前行有效，所以我们使用续行符来连接起来。应用续行符的时候要注意，在“\”后面不能有任何字符(包括注释、空格)，只能直接回车。

##### 3、LED GPIO初始化函数

编写LED灯的初始化函数

```c
/**
  * @brief  初始化控制LED的IO
  * @param  无
  * @retval 无
  */
void LED_GPIO_Config(void)
{		
		/*定义一个GPIO_InitTypeDef类型的结构体*/
		GPIO_InitTypeDef GPIO_InitStructure;

		/*开启LED相关的GPIO外设时钟*/
		RCC_APB2PeriphClockCmd( LED1_GPIO_CLK | LED2_GPIO_CLK , ENABLE);
		/*选择要控制的GPIO引脚*/
		GPIO_InitStructure.GPIO_Pin = LED1_GPIO_PIN;	

		/*设置引脚模式为通用推挽输出*/
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;   

		/*设置引脚速率为50MHz */   
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz; 

		/*调用库函数，初始化GPIO*/
		GPIO_Init(LED1_GPIO_PORT, &GPIO_InitStructure);	
		
		/*选择要控制的GPIO引脚*/
		GPIO_InitStructure.GPIO_Pin = LED2_GPIO_PIN;

		/*调用库函数，初始化GPIO*/
		GPIO_Init(LED2_GPIO_PORT, &GPIO_InitStructure);		

		/* 关闭所有led灯	*/
		GPIO_SetBits(LED1_GPIO_PORT, LED1_GPIO_PIN);
		
		/* 关闭所有led灯	*/
		GPIO_SetBits(LED2_GPIO_PORT, LED2_GPIO_PIN);
}
```

(1) 使用GPIO_InitTypeDef 定义GPIO 初始化结构体变量，以便下面用于存储GPIO 配置。
(2) 调用库函数RCC_APB2PeriphClockCmd 来使能LED 灯的GPIO 端口时钟，在前面的章节中我们是直接向RCC 寄存器赋值来使能时钟的，不如这样直观。该函数有两个输入参数，第一个参数用于指示要配置的时钟，如本例中的“RCC_ APB2Periph_GPIOC”，应用时我们使用“|”操作同时配置2 个LED 灯的时钟；函数的第二个参数用于设置状态，可输入“Disable”关闭或“Enable”使能时钟。
(3) 向GPIO 初始化结构体赋值，把引脚初始化成推挽输出模式，其中的GPIO_Pin 使用宏“LEDx_GPIO_PIN”来赋值，使函数的实现方便移植。
(4) 使用以上初始化结构体的配置，==调用GPIO_Init 函数向寄存器写入参数，完成GPIO 的初始化==，这里的GPIO 端口使用“LEDx_GPIO_PORT”宏来赋值，也是为了程序移植方便。
(5) 使用同样的初始化结构体，只修改控制的引脚和端口，初始化其它LED 灯使用的GPIO 引脚。
(6) 调用固件库函数控制LED 灯默认关闭。

##### 4、主函数

```c
#include "stm32f10x.h"
#include "bsp_led.h"

#define SOFT_DELAY Delay(0x0FFFFF);

void Delay(__IO u32 nCount); 

/**
  * @brief  主函数
  * @param  无  
  * @retval 无
  */
int main(void)
{	
	/* LED 端口初始化 */
	LED_GPIO_Config();	 

	while (1)
	{
		LED1_ON;			  // 亮
		SOFT_DELAY;
		LED1_OFF;		   // 灭

		LED2_ON;			 // 亮
		SOFT_DELAY;
		LED2_OFF;		   // 灭

	}
}

void Delay(__IO uint32_t nCount)	 //简单的延时函数
{
	for(; nCount != 0; nCount--);
}
```

#### 下载验证

### STM32标准库补充知识

#### 1、SystemInit函数去哪了？

在前面章节中我们自己建工程的时候需要定义一个SystemInit 空函数，但是在这个用STM32 标准库的工程却没有这样做，SystemInit 函数去哪了呢？
这个函数在STM32 标准库的“system_stm32f10x.c”文件中定义了，而我们的工程已经包含该文件。标准库中的SystemInit 函数把STM32 芯片的系统时钟设置成了72MHz，即此时AHB 时钟频率为72MHz，APB2 为72MHz，APB1 为36MHz。当STM32 芯片上电后，执行启动文件中的指令后，会调用该函数，设置系统时钟为以上状态。

#### 2、断言

细心对比过前几章我们自己定义的GPIO_Init 函数与STM32 标准库中同名函数的读者，会发现标准库中的函数内容多了一些乱七八糟的东西，就是断言

```c
void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct)
{
  uint32_t currentmode = 0x00, currentpin = 0x00, pinpos = 0x00, pos = 0x00;
  uint32_t tmpreg = 0x00, pinmask = 0x00;
  /* Check the parameters */
  assert_param(IS_GPIO_ALL_PERIPH(GPIOx));
  assert_param(IS_GPIO_MODE(GPIO_InitStruct->GPIO_Mode));
  assert_param(IS_GPIO_PIN(GPIO_InitStruct->GPIO_Pin));  
  
```

"assert_param"实际是一个宏，在库函数中它用于检查输入参数是否符合要求，若不符合要求则执行某个函数输出警告，“assert_param”的定义见代码。

```c
#ifdef  USE_FULL_ASSERT

/**
  * @brief  The assert_param macro is used for function's parameters check.
  			assert_param  宏用于函数的输入参数检查
  * @param  expr: If expr is false, it calls assert_failed function which reports   				  
  *         the name of the source file and the source line number of the call 
  *         that failed. If expr is true, it returns no value.
  			若expr值为假，则调用assert_failed函数
  			报告文件名及错误行号
  			若expr值为真，则不执行操作
  * @retval None
  */
  #define assert_param(expr) ((expr) ? (void)0 : assert_failed((uint8_t *)__FILE__, __LINE__))
/* Exported functions 错误输出函数------------------------------------------------------- */
void assert_failed(uint8_t* file, uint32_t line);
#else
  #define assert_param(expr) ((void)0)
#endif /* USE_FULL_ASSERT */
```

这段代码的意思是，假如我们不定义“USE_FULL_ASSERT”宏，那么“assert_param”就是一个空的宏(#else 与#endif 之间的语句生效)，没有任何操作。从而所有库函数中的assert_param 实际上都无意义，我们就当看不见好了。

假如我们定义了“USE_FULL_ASSERT”宏，那么“assert_param”就是一个有操作的语句(#if 与#else 之间的语句生效)，该宏对参数expr 使用C 语言中的问号表达式进行判断，若expr 值为真，则无操作(void 0)，若表达式的值为假，则调用“assert_failed”函数，且该函数的输入参数为“ __ FILE__ ”及 “ __ LINE__”，这两个参数分别代表 “assert_param”宏被调用时所在的“文件名”及“行号”。

但库文件只对“assert_failed”写了函数声明，没有写函数定义，实际用时需要用户来定义，我们一般会用printf 函数来输出这些信息。

```c
void assert_failed(uint8_t* file, uint32_t line)
{
    printf("\r\n 输入参数错误,错误文件名=%s,行号=%s,file,line);
}
```

注意在我们的这个LED 工程中，还不支持printf 函数(在USART 外设章节会讲解)，想测试assert_failed 输出的读者，可以在这个函数中做点亮红色LED 灯的操作，作为警告输出测试。
那么为什么函数输入参数不对的时候，assert_param 宏中的expr 参数值会是假呢？这要回到GPIO_Init 函数， 看它对assert_param 宏的调用， 它被调用时分别以“IS_GPIO_ALL_PERIPH(GPIOx)”、“IS_GPIO_PIN(GPIO_InitStruct->GPIO_Pin)”等作为输入参数，也就是说被调用时，expr 实际上是一条针对输入参数的判断表达式。例如“IS_GPIO_PIN”的宏定义：

```c
#define IS_GPIO_PIN(PIN)	((PIN) != (uint32_t)0x00)
```

若它的输入参数 PIN 值为0，则表达式的值为假，PIN 非0 时表达式的值为真。我们知道用于选择GPIO 引脚号的宏“GPIO_Pin_x”的值至少有一个数据位为1，这样的输入参数才有意义，若GPIO_InitStruct->GPIO_Pin 的值为0 ，输入参数就无效了。配合“ IS_GPIO_PIN”这句表达式，“ assert_param”就实现了检查输入参数的功能。对assert_param 宏的其它调用方式类似，大家可以自己看库源码来研究一下。

#### 3、Doxygen注释规范

百度自行学习

##### 4、防止头文件重复包含

```c
#ifndef __LED_H
#define __LED_H

/*此处省略头文件的具体内容*/

#endif /* end of __LED_H */
```

在头文件的开头，使用“#ifndef”关键字，判断标号“__ LED_H”是否被定义，若没有被定义，则从“#ifndef”至“#endif”关键字之间的内容都有效，也就是说，这个头文件若被其它文件“#include”，它就会被包含到其该文件中了，且头文件中紧接着使用“#define”关键字定义上面判断的标号“__ LED_H”。当这个头文件被同一个文件第二次“#include”包含的时候，由于有了第一次包含中的“#define __ LED_H”定义，这时再判断“#ifndef __LED_H”，判断的结果就是假了，从“#ifndef”至“#endif”之间的内容都无效，从而防止了同一个头文件被包含多次，编译时就不会出现“redefine（重复定义）”的错误了。
一般来说，我们不会直接在C 的源文件写两个“#include”来包含同一个头文件，但可能因为头文件内部的包含导致重复，这种代码主要是避免这样的问题。如“bsp_led.h”文件中使用了“#include ―stm32f10x.h‖ ”语句，按习惯，可能我们写主程序的时候会在main文件写“#include ―bsp_led.h‖ 及#include ―stm32f10x.h‖”，这个时候“stm32f10x.h”文件就被包含两次了，如果没有这种机制，就会出错。

至于为什么要用两个下划线来定义“__LED_H”标号，其实这只是防止它与其它普通宏定义重复了，如我们用“GPIO_PIN_0”来代替这个判断标号，就会因为stm32f10x.h 已经定义了GPIO_PIN_0，结果导致“bsp_led.h”文件无效了，“bsp_led.h”文件一次都没被包含。

## GPIO输入---按键检测

本章参考资料： 《STM32F10X- 中文参考手册》、库帮助文档《stm32f10x_stdperiph_lib_um》。

### 硬件设计

机械按键的触点断开、闭合时，由于触点的弹性作用，按键开关不会马上稳定接通或一下子断开，使用按键时会产生图中的带波纹信号，进行按键中消抖处理有两种办法，一种是硬件消抖，还有一种是软件消抖。

需要用软件消抖处理滤波，不方便输入检测。

本实验板连接的按键带硬件消抖功能，见图，它利用电容充放电的延时，消除了波纹，从而简化软件的处理，软件只需要直接检测引脚的电平即可。

![2](https://github.com/Leon199601/MCU/blob/main/pic/w1-2.jpg)

从按键的原理图可知，这些按键在没有被按下的时候，GPIO 引脚的输入状态为低电平(按键所在的电路不通，引脚接地)，当按键按下时，GPIO 引脚的输入状态为高电平(按键所在的电路导通，引脚接到电源)。只要我们检测引脚的输入电平，即可判断按键是否被按下。

若您使用的实验板按键的连接方式或引脚不一样，只需根据我们的工程修改引脚即可，程序的控制原理相同。

### 软件设计

方便移植，在“工程模板”上新建“bsp_key.c"及"bsp_key.h"文件

#### 编程要点

1.使能GPIO端口时钟

2.初始化GPIO目标引脚为输入模式（浮空输入）

3.编写简单测试程序，检测按键状态，实现按键控制LED灯

#### 代码分析

##### 1、按键引脚宏定义

编写按键驱动时，考虑更改硬件环境，要把按键检测引脚宏定义写到“bsp_key.h”

```c
//  引脚定义
#define    KEY1_GPIO_CLK     RCC_APB2Periph_GPIOA
#define    KEY1_GPIO_PORT    GPIOA			   
#define    KEY1_GPIO_PIN		 GPIO_Pin_0

#define    KEY2_GPIO_CLK     RCC_APB2Periph_GPIOC
#define    KEY2_GPIO_PORT    GPIOC		   
#define    KEY2_GPIO_PIN		  GPIO_Pin_13


 /** 按键按下标置宏
	*  按键按下为高电平，设置 KEY_ON=1， KEY_OFF=0
	*  若按键按下为低电平，把宏设置成KEY_ON=0 ，KEY_OFF=1 即可
	*/
#define KEY_ON	1
#define KEY_OFF	0
```

将检测按键所需要的GPIO端口、引脚号以及端口时钟封装起来。

##### 2、按键GPIO初始化函数

```c
/**
  * @brief  配置按键用到的I/O口
  * @param  无
  * @retval 无
  */
void Key_GPIO_Config(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;
	
	/*开启按键端口的时钟*/
	RCC_APB2PeriphClockCmd(KEY1_GPIO_CLK|KEY2_GPIO_CLK,ENABLE);
	
	//选择按键的引脚
	GPIO_InitStructure.GPIO_Pin = KEY1_GPIO_PIN; 
	// 设置按键的引脚为浮空输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; 
	//使用结构体初始化按键
	GPIO_Init(KEY1_GPIO_PORT, &GPIO_InitStructure);
	
	//选择按键的引脚
	GPIO_InitStructure.GPIO_Pin = KEY2_GPIO_PIN; 
	//设置按键的引脚为浮空输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; 
	//使用结构体初始化按键
	GPIO_Init(KEY2_GPIO_PORT, &GPIO_InitStructure);	
}
```

(1)使用GPIO_InitTypeDef定义GPIO初始化结构体变量，以便下面用于存储GPIO配置

(2)调用库函数使能GPIO端口时钟

(3)向GPIO初始化结构体赋值，引脚设为浮空输入模式。由于引脚的默认电平受按键电路影响，所以设置为浮空输入。

(4)使用上面初始化结构体的配置，==调用GPIO_Init函数向寄存器写入参数，完成GPIO初始化。==

(5)使用同样的初始化结构体，只修改控制的引脚和端口，初始化其它按键检测时使用的GPIO 引脚。

##### 3、检测按键的状态

检测对应引脚的电平来判断按键状态

```
/*
 * 函数名：Key_Scan
 * 描述  ：检测是否有按键按下
 * 输入  ：GPIOx：x 可以是 A，B，C，D或者 E
 *		     GPIO_Pin：待读取的端口位 	
 * 输出  ：KEY_OFF(没按下按键)、KEY_ON（按下按键）
 */
uint8_t Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_Pin)
{			
	/*检测是否有按键按下 */
	if(GPIO_ReadInputDataBit(GPIOx,GPIO_Pin) == KEY_ON )  
	{	 
		/*等待按键释放 */
		while(GPIO_ReadInputDataBit(GPIOx,GPIO_Pin) == KEY_ON);   
		return 	KEY_ON;	 
	}
	else
		return KEY_OFF;
}
```

在这里定义了一个Key_Scan 函数用于扫描按键状态。GPIO 引脚的输入电平可通过读取IDR 寄存器对应的数据位来感知， 而STM32 标准库提供了库函数GPIO_ReadInputDataBit 来获取位状态，该函数输入GPIO 端口及引脚号，函数返回该引脚的电平状态，高电平返回1，低电平返回0。Key_Scan 函数中以GPIO_ReadInputDataBit 的返回值与自定义的宏“KEY_ON”对比，若检测到按键按下，则使用while 循环持续检测按键状态，直到按键释放，按键释放后Key_Scan 函数返回一个“KEY_ON”值；若没有检测到按键按下，则函数直接返回“KEY_OFF”。

若按键的硬件没有做消抖处理，需要在这个Key_Scan 函数中做软件滤波，防止波纹抖动引起误触发。

##### 4、主函数

```c
/**
  * @brief  主函数
  * @param  无
  * @retval 无
  */ 
int main(void)
{	
	/* LED端口初始化 */
	LED_GPIO_Config();
	LED1_ON;

	/* 按键端口初始化 */
	Key_GPIO_Config();
	
	/* 轮询按键状态，若按键按下则反转LED */
	while(1)                            
	{	   
		if( Key_Scan(KEY1_GPIO_PORT,KEY1_GPIO_PIN) == KEY_ON  )
		{
			/*LED1反转*/
			LED1_TOGGLE;
		} 

		if( Key_Scan(KEY2_GPIO_PORT,KEY2_GPIO_PIN) == KEY_ON  )
		{
			/*LED2反转*/
			LED2_TOGGLE;
		}		
	}
}
```

代码中初始化LED 灯及按键后，在while 函数里不断调用Key_Scan 函数，并判断其返回值，若返回值表示按键按下，则反转LED 灯的状态。

#### 下载验证

## GPIO---位带操作

本章参考资料：《STM32F10X-中文参考手册》存储器和总线构架章节、GPIO 章节，《CM3 权威指南CnR2》存储器系统章节。学习本章时，配套这些参考资料学习效果会更佳。

### 位带简介

位操作是单独对一个比特位读和写，stm32是通过访问位带别名区来实现的

stm32中，有两个地方实现位带，SRAM区的最低1MB空间，另一个是外设区最低1MB空间。这两个1MB 的空间除了可以像正常的RAM 一样操作外，他们还有自己的位带别名区，位带别名区把这1MB 的空间的每一个位膨胀成一个32 位的字，当访问位带别名区的这些字时，就可以达到访问位带区某个比特位的目的。

![3](https://github.com/Leon199601/MCU/blob/main/pic/w1-3.jpg)

#### 外设位带区

外设外带区的地址为：0X40000000 ~ 0X40100000，这个大小为1MB，这1MB 的大小在103 系列大/中/小容量型号的单片机中包含了片上外设的全部寄存器，这些寄存器的地址为： 0X40000000 ~ 0X40029FFF 。外设位带区经过膨胀后的位带别名区地址为：
0X42000000 ~ 0X43FFFFFF，这个地址仍然在CM3 片上外设的地址空间中。在103 系列大/中小容量型号的单片机里面，0X40030000 ~ 0X4FFFFFFF 属于保留地址，膨胀后的32MB位带别名区刚好就落到这个地址范围内，不会跟片上外设的其他寄存器地址重合。STM32 的全部寄存器都可以通过访问位带别名区的方式来达到访问原始寄存器比特位的效果，这比51 单片机强大很多。因为51 单片机里面并不是所有的寄存器都是可以比特位操作，有些寄存器还是得字节操作，比如SBUF。虽然说全部寄存器都可以实现比特操作，但我们在实际项目中并不会这么做，甚至不会这么做。有时候为了特定的项目需要，比如需要频繁的操作很多IO 口，这个时候我们可以考虑把IO 相关的寄存器实现比特操作。

#### SRAM位带区

SRAM的位带区的地址为：0X2000 0000 ~ X2010 0000，大小为1MB，经过膨胀后的位带别名区地址为：0X2200 0000 ~ 0X23FF FFFF，大小为32MB。操作SRAM 的比特位这个用得很少。

#### 位带区和位带别名区地址转换

位带区的一个比特位经过膨胀之后，虽然变大到4 个字节，但是还是LSB（最低位） 才有效。有人会问这不是浪费空间吗，要知道STM32 的系统总线是32 位的，按照4 个字节访问的时候是最快的，所以膨胀成4 个字节来访问是最高效的。
通过指针的形式访问位带别名区地址从而达到操作位带区比特位的效果。那这两个地址直接如何转换，简单介绍一下。

##### 外设位带别名区地址

对于片上外设位带区的某个比特，记它所在字节的地址为 A,位序号为 n(0<=n<=7)，则该比特在别名区的地址为：

```c
AliasAddr= =0x40000000+ （A-0x40000000）*8*4 +n*4
```

0X42000000 是外设位带别名区的起始地址，0x40000000是外设位带区的起始地址，（A-0x40000000）表示该比特前面有多少个字节，一个字节有8位，所以*8，一个位膨胀后是4个字节，所以*4，n表示该比特在A地址的序号，因为一个位经过膨胀后是四个字节，所以也*4。

##### SRAM位带别名区地址

对于SRAM 位带区的某个比特，记它所在字节的地址为 A,位序号为 n(0<=n<=7)，则该比特在别名区的地址为：

```c
AliasAddr= =0x22000000+ （A-0x20000000）*8*4 +n*4
```

##### 统一公式

为了方便，统一成一个宏

```c
// 把“位带地址+位序号”转换成别名地址的宏
#define BITBAND(addr, bitnum) ((addr & 0xF0000000)+0x02000000+((addr &
0x00FFFFFF)<<5)+(bitnum<<2))
```

addr & 0xF0000000 是为了区别SRAM还是外设，实际效果就是取出4 或者2，如果是外设，则取出的是4，+0X02000000 之后就等于0X42000000，0X42000000 是外设别名区的起始地址。如果是SRAM，则取出的是2，+0X02000000 之后就等于0X22000000，0X22000000 是SRAM别名区的起始地址。

addr & 0x00FFFFFF 屏蔽了高三位，相当于减去0X20000000 或者0X40000000，但是为什么是屏蔽高三位？因为外设的最高地址是：0X2010 0000，跟起始地址0X20000000 相减的时候，总是低5 位才有效，所以干脆就把高三位屏蔽掉来达到减去起始地址的效果，具体屏蔽掉多少位跟最高地址有关。SRAM 同理分析即可。<<5 相当于*8*4，<<2 相当于*4，这两个我们在上面分析过。

最后就可以通过指针的形式操作这些位带别名区地址，最终实现位带区的比特位操作。

```c
// 把一个地址转换成一个指针
#define MEM_ADDR(addr) *((volatile unsigned long *)(addr))

// 把位带别名区地址转换成指针
#define BIT_ADDR(addr, bitnum) MEM_ADDR(BITBAND(addr, bitnum))
```

### GPIO位带操作

外设的位带区，覆盖了全部的片上外设的寄存器，我们可以通过宏为每个寄存器的位都定义一个位带别名地址，从而实现位操作。但这个在实际项目中不是很现实，也很少人会这么做，我们在这里仅仅演示下GPIO 中ODR 和IDR 这两个寄存器的位操作。

从手册中我们可以知道ODR 和IDR 这两个寄存器对应GPIO 基址的偏移是12 和8，先实现这两个寄存器的地址映射，其中GPIOx_BASE在库函数里面有定义。

#### GPIO寄存器映射

```c
// GPIO ODR 和 IDR 寄存器地址映射 
#define GPIOA_ODR_Addr    (GPIOA_BASE+12) //0x4001080C   
#define GPIOB_ODR_Addr    (GPIOB_BASE+12) //0x40010C0C   
#define GPIOC_ODR_Addr    (GPIOC_BASE+12) //0x4001100C   
#define GPIOD_ODR_Addr    (GPIOD_BASE+12) //0x4001140C   
#define GPIOE_ODR_Addr    (GPIOE_BASE+12) //0x4001180C   
#define GPIOF_ODR_Addr    (GPIOF_BASE+12) //0x40011A0C      
#define GPIOG_ODR_Addr    (GPIOG_BASE+12) //0x40011E0C      
  
#define GPIOA_IDR_Addr    (GPIOA_BASE+8)  //0x40010808   
#define GPIOB_IDR_Addr    (GPIOB_BASE+8)  //0x40010C08   
#define GPIOC_IDR_Addr    (GPIOC_BASE+8)  //0x40011008   
#define GPIOD_IDR_Addr    (GPIOD_BASE+8)  //0x40011408   
#define GPIOE_IDR_Addr    (GPIOE_BASE+8)  //0x40011808   
#define GPIOF_IDR_Addr    (GPIOF_BASE+8)  //0x40011A08   
#define GPIOG_IDR_Addr    (GPIOG_BASE+8)  //0x40011E08 
```

现在就可以用位操作的方法来控制GPIO 的输入和输出了，其中宏参数n 表示具体是哪一个IO 口，n(0,1,2...16)。这里面包含了端口A~G，并不是每个单片机型号都有这么多端口，使用这部分代码时，要查看你的单片机型号，如果是64pin 的则最多只能使用到C 端口。

#### GPIO位操作

```c
// 单独操作 GPIO的某一个IO口，n(0,1,2...16),n表示具体是哪一个IO口
#define PAout(n)   BIT_ADDR(GPIOA_ODR_Addr,n)  //输出   
#define PAin(n)    BIT_ADDR(GPIOA_IDR_Addr,n)  //输入   
  
#define PBout(n)   BIT_ADDR(GPIOB_ODR_Addr,n)  //输出   
#define PBin(n)    BIT_ADDR(GPIOB_IDR_Addr,n)  //输入   
  
#define PCout(n)   BIT_ADDR(GPIOC_ODR_Addr,n)  //输出   
#define PCin(n)    BIT_ADDR(GPIOC_IDR_Addr,n)  //输入   
  
#define PDout(n)   BIT_ADDR(GPIOD_ODR_Addr,n)  //输出   
#define PDin(n)    BIT_ADDR(GPIOD_IDR_Addr,n)  //输入   
  
#define PEout(n)   BIT_ADDR(GPIOE_ODR_Addr,n)  //输出   
#define PEin(n)    BIT_ADDR(GPIOE_IDR_Addr,n)  //输入  
  
#define PFout(n)   BIT_ADDR(GPIOF_ODR_Addr,n)  //输出   
#define PFin(n)    BIT_ADDR(GPIOF_IDR_Addr,n)  //输入  
  
#define PGout(n)   BIT_ADDR(GPIOG_ODR_Addr,n)  //输出   
#define PGin(n)    BIT_ADDR(GPIOG_IDR_Addr,n)  //输入 
```

#### 主函数

```c
int main(void)
{	
	// 程序来到main函数之前，启动文件：statup_stm32f10x_hd.s已经调用
	// SystemInit()函数把系统时钟初始化成72MHZ
	// SystemInit()在system_stm32f10x.c中定义
	// 如果用户想修改系统时钟，可自行编写程序修改
	
	LED_GPIO_Config();
	
	while( 1 )
	{
		// PC2 = 0,PC3 = 0,点亮LED
		PCout(2)= 0;	
		PCout(3)= 0;			
		SOFT_Delay(0x0FFFFF);
		
		// PC2 = 1,PC3 = 1,熄灭LED		
		PCout(2)= 1;
		PCout(3)= 1;
		SOFT_Delay(0x0FFFFF);		
	}
}
```

#### 下载验证

## 启动文件详解

本章参考资料《STM32F10X-中文参考手册》第九章-中断和事件：表55其他STM32F10xxx 产品（小容量、中容量和大容量）的向量表；MDK 中的帮助手册—ARM Development Tools：用来查询ARM的汇编指令和编译器相关的指令。

### 启动文件简介

启动文件由汇编编写，，是系统上电复位后第一个执行的程序。主要做了以下工作：
1、初始化堆栈指针 SP = _initial_sp
2、初始化PC 指针 = Reset_Handler
3、初始化中断向量表
4、配置系统时钟
5、调用C 库函数 _main 初始化用户堆栈，从而最终调用main 函数去到C 的世界

### 查看ARM汇编指令

在讲解启动代码的时候，会涉及到ARM 的汇编指令和Cortex 内核的指令，有关Cortex 内核的指令我们可以参考《CM3 权威指南CnR2》第四章：指令集。剩下的ARM的汇编指令我们可以在MDK->Help->Uvision Help 中搜索到，以EQU 为例，检索如下：

![4](https://github.com/Leon199601/MCU/blob/main/pic/w1-4.jpg)

检索出来的结果会有很多，只需要看Assembler User Guide 这部分即可。列出了启动文件中使用到的ARM 汇编指令，该列表的指令全部从ARM Development Tools这个帮助文档里面检索而来。

其中编译器相关的指令WEAK 和ALIGN 为了方便也放在同一个表格了。

| 指令名称       | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| EQU            | 给数字常量取一个符号名，相当于C 语言中的define               |
| AREA           | 汇编一个新的代码段或者数据段                                 |
| SPACE          | 分配内存空间                                                 |
| PRESERVE8      | 当前文件堆栈需按照8 字节对齐                                 |
| EXPORT         | 声明一个标号具有全局属性，可被外部的文件使用                 |
| DCD            | 以字为单位分配内存，要求4 字节对齐，并要求初始化这些内存     |
| PROC           | 定义子程序，与ENDP 成对使用，表示子程序结束                  |
| WEAK           | 弱定义，如果外部文件声明了一个标号，则优先使用外部文件定义的标号，如果外部文件没有定义也不出错。要注意的是：这个不是ARM的指令，是编译器的，这里放在一起只是为了方便。 |
| IMPORT         | 声明标号来自外部文件，跟C 语言中的EXTERN 关键字类似          |
| B              | 跳转到一个标号                                               |
| ALIGN          | 编译器对指令或者数据的存放地址进行对齐，一般需要跟一个立即数，缺省表示4字节对齐。要注意的是：这个不是ARM的指令，是编译器的，这里放在一起只是为了方便。 |
| END            | 到达文件的末尾，文件结束                                     |
| IF,EL,SE,ENDIF | 汇编条件分支语句，跟C语言的if else类似                       |

### 启动文件代码讲解

#### Stack---栈

```assembly
Stack_Size		EQU		0x00000400

				AREA	STACK,	NOINIT,	READWRITE,	ALIGN=3
Stack_Mem		SPACE	Stack_Size
__initial_sp
```

开辟栈的大小为0X00000400（1KB），名字为STACK，NOINIT 即不初始化，可读可写，8（2^3）字节对齐。
栈的作用是用于局部变量，函数调用，函数形参等的开销，栈的大小不能超过内部SRAM的大小。如果编写的程序比较大，定义的局部变量很多，那么就需要修改栈的大小。如果某一天，你写的程序出现了莫名奇怪的错误，并进入了硬fault的时候，这时你就要考虑下是不是栈不够大，溢出了。

EQU：宏定义的伪指令，相当于等于，类似与C 中的define。
AREA：告诉汇编器汇编一个新的代码段或者数据段。STACK 表示段名，这个可以任意命名；NOINIT 表示不初始化；READWRITE 表示可读可写，ALIGN=3，表示按照2^3对齐，即8 字节对齐。
SPACE：用于分配一定大小的内存空间，单位为字节。这里指定大小等于Stack_Size。标号__initial_sp 紧挨着SPACE 语句放置，表示栈的结束地址，即栈顶地址，栈是由高向低生长的。

#### Heap堆

```assembly
Heap_Size		EQU		0x00000200

				AREA	HRAP,NOINIT,READWRITE,ALIGN=3
__heap_base
Heap_Mem		SPACE	Heap_Size
__heap_limit
```

开辟堆的大小为0X00000200（512 字节），名字为HEAP，NOINIT 即不初始化，可读可写，8（2^3）字节对齐。__ heap_base 表示对的起始地址，__heap_limit 表示堆的结束地址。堆是由低向高生长的，跟栈的生长方向相反。
堆主要用来动态内存的分配，像malloc()函数申请的内存就在堆上面。这个在STM32里面用的比较少。

```assembly
PRESERVE8
THUMB
```

PRESERVE8：指定当前文件的堆栈按照8 字节对齐。
THUMB：表示后面指令兼容THUMB 指令。THUBM是ARM以前的指令集，16bit，现在Cortex-M系列的都使用THUMB-2 指令集，THUMB-2 是32 位的，兼容16 位和32 位的指令，是THUMB 的超集。

#### 向量表

```assembly
AREA		RESET,DATA,READONLY
EXPORT	__Vectors
EXPORT	__Vectors_End
EXPORT	__Vectors_Size
```

定义一个数据段，名字为RESET，可读。并声明 __ Vectors、__ Vectors_End 和__ Vectors_Size 这三个标号具有全局属性，可供外部的文件调用。
EXPORT：声明一个标号可被外部的文件使用，使标号具有全局属性。如果是IAR 编译器，则使用的是GLOBAL 这个指令。
当内核响应了一个发生的异常后，对应的异常服务例程(ESR)就会执行。为了决定 ESR的入口地址， 内核使用了―向量表查表机制‖。这里使用一张向量表。向量表其实是一个WORD（ 32 位整数）数组，每个下标对应一种异常，该下标元素的值则是该 ESR 的入口地址。向量表在地址空间中的位置是可以设置的，通过 NVIC 中的一个重定位寄存器来指出向量表的地址。在复位后，该寄存器的值为 0。因此，在地址 0 （即FLASH 地址0）处必须包含一张向量表，用于初始时的异常分配。要注意的是这里有个另类： 0 号类型并不是什么入口地址，而是给出了复位后 MSP 的初值。

![5](https://github.com/Leon199601/MCU/blob/main/pic/w1-5.jpg)

```assembly
__Vectors       DCD     __initial_sp               ; Top of Stack ---栈顶地址
                DCD     Reset_Handler              ; Reset Handler---复位程序地址
                DCD     NMI_Handler                ; NMI Handler
                DCD     HardFault_Handler          ; Hard Fault Handler
                DCD     MemManage_Handler          ; MPU Fault Handler
                DCD     BusFault_Handler           ; Bus Fault Handler
                DCD     UsageFault_Handler         ; Usage Fault Handler
                DCD     0                          ; Reserved		0 表示保留
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     SVC_Handler                ; SVCall Handler
                DCD     DebugMon_Handler           ; Debug Monitor Handler
                DCD     0                          ; Reserved
                DCD     PendSV_Handler             ; PendSV Handler
                DCD     SysTick_Handler            ; SysTick Handler

;外部中断开始
				DCD     WWDG_IRQHandler            ; Window Watchdog
                DCD     PVD_IRQHandler             ; PVD through EXTI Line detect
                DCD     TAMPER_IRQHandler          ; Tamper
                
;限于篇幅，中间代码省略
DCD     DMA2_Channel2_IRQHandler   ; DMA2 Channel2
                DCD     DMA2_Channel3_IRQHandler   ; DMA2 Channel3
                DCD     DMA2_Channel4_5_IRQHandler ; DMA2 Channel4 & Channel5
__Vectors_End

__Vectors_Size  EQU  __Vectors_End - __Vectors
```

__ Vectors 为向量表起始地址，__Vectors_End为向量表结束地址，两个相减即可算出向量表大小。
向量表从FLASH 的0 地址开始放置，以4 个字节为一个单位，地址0 存放的是栈顶地址，0X04 存放的是复位程序的地址，以此类推。从代码上看，向量表中存放的都是中断服务函数的函数名，可我们知道C 语言中的函数名就是一个地址。
DCD：分配一个或者多个以字为单位的内存，以四字节对齐，并要求初始化这些内存。在向量表中，DCD 分配了一堆内存，并且以ESR 的入口地址初始化它们。

#### 复位程序

```assembly
AREA	|.text|,	CODE,	READONLY
```

定义一个名称为.text的代码段，可读。

```assembly
Reset_Handler PROC
			  EXPORT	Reset_Handle	[WEAK]
			  IMPORT	SystemInit
			  IMPORT	__mian
			  
			  LDR	R0,=SystemInit
			  BLX	R0
			  LDR	R0，=__MAIN
			  BX	R0
			  ENDP
```

复位子程序是系统上电后第一个执行的程序，调用SystemInit函数初始化系统时钟，然后调用C库函数_main,最终调用main函数去到C的世界。

WEAK：表示弱定义，如果外部文件优先定义了该标号则首先引用该标号，如果外部文件没有声明也不会出错。这里表示复位子程序可以由用户在其他文件重新实现，这里并不是唯一的。
IMPORT：表示该标号来自外部文件，跟C 语言中的EXTERN 关键字类似。这里表示SystemInit __ main 这两个函数均来自外部的文件。
SystemInit()是一个标准的库函数，在system_stm32f10x.c 这个库文件总定义。主要作用是配置系统时钟，这里调用这个函数之后，单片机的系统时钟配被配置为72M。
__main 是一个标准的C 库函数，主要作用是初始化用户堆栈，并在函数的最后调用main 函数去到C 的世界。这就是为什么我们写的程序都有一个main 函数的原因。

LDR、BLX、BX是CM4内核的指令，可在《CM3权威指南CnR2》第四章-指令集里面查询到，具体作用见表：

| 指令名称 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| LDR      | 从存储器中加载字到一个寄存器中                               |
| BL       | 跳转到由寄存器/标号给出的地址，并把跳转前的下条指令地址保存到LR |
| BLX      | 跳转到由寄存器给出的地址，并根据寄存器的LSE 确定处理器的状态，还要<br/>把跳转前的下条指令地址保存到LR |
| BX       | 跳转到由寄存器/标号给出的地址，不用返回                      |

#### 中断服务程序

在启动文件里面已经写好了所有中断的中断服务函数，跟平时写的中断服务函数不一样的就是这些函数是空的，真正的中断服务程序需要在外部的C文件里面重新实现，这里只是提前占了一个位置而已。

如果使用某个外设的时候，开启了某个中断，但是又忘记编写配套的中断服务程序或者函数名写错，那当中断来临的时，程序会跳转到启动文件预先写好的空的中断服务函数中，并且在这个空函数中无限循环，即程序就是死在这了。

```assembly
NMT_Handler		PROC	;系统异常
				EXPORT	NMI_Handler				[WEAK]
				B		.
				ENDP
				
;限于篇幅，中间代码省略
SysTick_Handler PROC
				EXPORT	SysTick_Handler			[WEAK]
				B		.
				ENDP
				
Default_Handler PROC	;外部中断
				EXPORT	WWDG_IRQHandler			[WEAK]
				EXPORT	PVD_IRQHandler			[WEAK]
				EXPORT	TAMP_STAMP_IRQHandler	[WEAK]
				
;限于篇幅，中间代码省略
LTDC_IRQHandler
LTDC_ER_IRQHandler
DMA2D_IRQHandler
				B		.
				ENDP
```

B:跳转到一个标号。这里跳转到一个’.‘，即表示无限循环。

#### 用户堆栈初始化

ALIGN:对指令或者数据存放的地址进行对齐，后面会跟一个立即数。缺省表示4字节对齐。

```assembly
; User Stack and Heap initialization
;用户栈和堆初始化，由C库函数_main来完成
                 IF      :DEF:__MICROLIB	;这个宏在KEIL里面启动
                
                 EXPORT  __initial_sp
                 EXPORT  __heap_base
                 EXPORT  __heap_limit
                
                 ELSE
                
                 IMPORT  __use_two_region_memory	;这个函数由用户自己实现
                 EXPORT  __user_initial_stackheap
                 
__user_initial_stackheap

                 LDR     R0, =  Heap_Mem
                 LDR     R1, =(Stack_Mem + Stack_Size)
                 LDR     R2, = (Heap_Mem +  Heap_Size)
                 LDR     R3, = Stack_Mem
                 BX      LR

                 ALIGN

                 ENDIF

                 END
```

首先判断是否定义了 _ _  MICROLIB，如果定义了这个宏则赋予标号   _ _ initial_sp（栈顶地址）、_ _ heap_base（堆起始地址）、_ _ heap_limit（堆结束地址）全局属性，可供外部文件调用。有关这个宏我们在KEIL 里面配置，具体见下图。然后堆栈的初始化就由C 库
函数_main 来完成。

![6](https://github.com/Leon199601/MCU/blob/main/pic/w1-6.jpg)

如果没有定义__ MICROLIB ， 则才用双段存储器模式， 且声明标号__user_initial_stackheap 具有全局属性，让用户自己来初始化堆栈。
IF,ELSE,ENDIF：汇编的条件分支语句，跟C 语言的if ,else 类似
END：文件结束

## RCC---使用HSE/HSI配置时钟
