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
(4) 使用以上初始化结构体的配置，调用GPIO_Init 函数向寄存器写入参数，完成GPIO 的初始化，这里的GPIO 端口使用“LEDx_GPIO_PORT”宏来赋值，也是为了程序移植方便。
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

机械按键的触点断开、闭合时，由于触点的弹性作用，按键开关不会马上稳定接通或一下子断开，使用按键时会产生图中的带波纹信号，需要用软件消抖处理滤波，不方便输入检测。本实验板连接的按键带硬件消抖功能，见图，它利用电容充放电的延时，消除了波纹，从而简化软件的处理，软件只需要直接检测引脚的电平即可。

![2](https://github.com/Leon199601/MCU/blob/main/pic/w1-2.jpg)

从按键的原理图可知，这些按键在没有被按下的时候，GPIO 引脚的输入状态为低电平(按键所在的电路不通，引脚接地)，当按键按下时，GPIO 引脚的输入状态为高电平(按键所在的电路导通，引脚接到电源)。只要我们检测引脚的输入电平，即可判断按键是否被按下。

若您使用的实验板按键的连接方式或引脚不一样，只需根据我们的工程修改引脚即可，程序的控制原理相同。

### 软件设计

