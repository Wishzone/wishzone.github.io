---
title: STM32
date: 2022-12-10 22:18:36
tags: STM32
categories: 单片机
---
# 配置

* vscode 插件 Keil Assistant
* keil启动vscode

  * 编码改为UTF-8，Tab-size为4
  * 在Edit->Configuration->Editor->File&Project Handing里勾上了Automatic reload of externally modified files
  * 点击mdk菜单栏Tools->Customize Tools Menu，在弹出的对话框中新建一个外部编辑器，并指定其路径， **注意Arguments要填上#E** ，我的vscode路径：D:\Program Files\Microsoft VS CodeCode.exe
  * 点击确定后，点击mdk菜单栏Tools->VSCode，即可自动调用vscode打开当前文件
* [Keil仿真监控数据导出到EXCEL图表显示的方法](https://blog.csdn.net/asdfghjkl1234567890p/article/details/125522052)
* [Keil5把变量的数据导出，可视化](https://blog.csdn.net/QXF0806/article/details/125687203)经验

1. switch -case语句中 case: 后面不能直接定义变量，但是可以先写一个；来解决这个问题
2. [Nucleo-G474RE作为烧写器](https://zhuanlan.zhihu.com/p/111723881)
3. [理解 LCD 屏幕的驱动原理与调试过程，示例的驱动 IC 为 GC9308 ，展示整个屏幕的驱动过程。](http://t.zoukankan.com/juwan-p-13069102.html)

# 时钟

## SysTick定时时间计算

t = reload *(1/clk)
clk = 72m时 t = (72)*(1/72M) = 1us
clk = 72m时 t = (72000)* (1/72M) = 1ms
1s = 1000ms = 1000 000us = 1000 000 000ns
记得使用 `HAL_TIM_Base_Start(&htim1);`

# keil 软件仿真

<!--more-->

## stm32f103VET6

1. debug页面如下设置
[![KEIL_DEBUGKEIL_DEBUG](https://shjdgwj.github.io/e6640d881469/images/loading.gif)](https://shjdgwj.github.io/e6640d881469/1.png)
2. 打开逻辑分析仪
3. setup打开后如下设置[![](https://shjdgwj.github.io/e6640d881469/images/loading.gif)](https://shjdgwj.github.io/e6640d881469/2.png)
   填 `PORTB.0`即可查看GPIOB pin0的输出
   `DisPlay Type` 选择 `Bit`
   之后就能输出PWM波形了

## stm32f103C8T6

[软件仿真配置](https://blog.csdn.net/keygun/article/details/97619613)
CPU
DLL：SARMCM3.DLL

Dialog
DLL：DARMSTM.DLL Parameter:-pSTM32F103C8

Driver
DLL SARMCM3.DLL

Dalog
DLL:TARMSTM.DLL Parameter:-pSTM32F103C8

# C相关

## STM32使用malloc函数

对于malloc和free对内存堆栈块的空间操作，在keilMDK中需要满足下面几个条件：

1. 使用的代码文件中需要包含头文件 `#include <stdlib.h>`
2. 在工程的属性设置中需要把 Use MicroLIB 选项勾选
3. 这时候原则上就可以使用空间申请和释放的两个操作函数了，但是由于STM32在startup_stm32f10x_hd.s中分配的堆空间只有0x00000200个字节，所以很多时候调用malloc函数时如果申请空间超过0X200则返回了NULL，这时候就需要到该文件对这个值进行设置。

## 单片机中的数据类型

u8——1个字节，无符号型（不能表达负数，如果用来当作负数的话，就出错了）
u16 ——2个字节，无符号型（参看前边STM32f10x.h中的定义）
u32——4个字节，无符号型
int——4个字节,有符号型，可以表达负整数
float ——4个字节，有符号型，可以表达负数/小数
double——8个字节，有符号弄，可以表达负数/小数

## 代码书写规范

变量定义在.c 在.h中用extern供外部引用
宏定义直接define在.h最前面
函数定义在.c 在.h中再写一遍名字即可
结构体和枚举需要将类型名定义在 .h ，将变量定义在.c 在 .h中用extern引用

## while(*str)的含义

字符串是以 ‘\0’结束的 当 指针 s指向最后一个 即是 ‘\0’是 *s=’\0’ 也等于 0 即是假的 结束循环

## 判断char数组里是否是汉字

百度说

> 负数是为汉字，二个字节一个
> gb2312 两个字节都是负的
> 如果是gbk，第一个字节还是负的，第二字节就不一定了

 **但是** ！不知道为什么，在keil中

```C++
char* str;
while(*str)
    {
        if(*str >= 0&&str<128){
            ...
        }else{
            //反正非汉字字符一定在0-127间
            ...
        }
    }
```

`str<128`的部分会被keil提示成 `always true`,而实际仿真中，变量窗口识别str为uchar，也就是中文的值并非<0而是>127。。。
改进后：

```C++
while(*str)
    {
  uint8_t ch = *str;
        if(ch >= 0&&ch<128)
        {
            showchar(*str, color);
            str++;
        }
        else//汉字2个char 第一个char<0
        {
            char s8[2];
   int16_t *s16 ;
   s16 = (int16_t*)str;
   str +=2;
            showChinese(*s16, color);
        }
    }
```

## 其他

无符号与有符号数进行运算时，系统会自动将有符号数看成无符号数，然后进行比较。举例:
假定一个数是8位，一个字节表示
-1=11111111(补码)，相当于无符号数255
10=00001010，此时，-1>10

# 外设

## TIM定时器

计数器时钟 CK_CNT = CK_PSC/(PSC+1)
计数一次的时间 1/CK_CNT
周期 1/CK_CNT*(ARR+1)

### 输入捕获

[stm32f103输入捕获](https://blog.csdn.net/zj490044512/article/details/83754414)
[【STM32】HAL库 STM32CubeMX教程八—定时器输入捕获](https://blog.csdn.net/as480133937/article/details/99407485)
[使用 STM32 测量频率和占空比的几种方法](https://blog.csdn.net/yyx112358/article/details/78414594)
[STM32F1x HAL库学习笔记（11）定时器配置及中断（溢出中断，PWM输出，输入捕获）](https://blog.csdn.net/qq_17351161/article/details/107386857)

### 输出比较

```C++
/* Blocking mode: Polling 轮询模式*/

HAL_StatusTypeDef HAL_TIM_OC_Start(TIM_HandleTypeDef *htim, uint32_t Channel);
HAL_StatusTypeDef HAL_TIM_OC_Stop(TIM_HandleTypeDef htim, uint32_t Channel);
/* Non-Blocking mode: Interrupt 中断模式 */

HAL_StatusTypeDef HAL_TIM_OC_Start_IT(TIM_HandleTypeDef htim, uint32_t Channel);
HAL_StatusTypeDef HAL_TIM_OC_Stop_IT(TIM_HandleTypeDef htim, uint32_t Channel);
/* Non-Blocking mode: DMA DMA模式*/

HAL_StatusTypeDef HAL_TIM_OC_Start_DMA(TIM_HandleTypeDef *htim, uint32_t Channel, uint32_t *pData, uint16_t Length);
HAL_StatusTypeDef HAL_TIM_OC_Stop_DMA(TIM_HandleTypeDef *htim, uint32_t Channel);
```

### 全速运行卡死在-HAL_TIM_Base_Start_IT函数

> 我没有猜错的话 你的是NVIC撞车了 都是 0 0 滴答定时器就阵亡了 修改一下分配的优先级就好了

### HAL库微秒级延时

#### 优选-获取系统时钟计时，非阻塞式延时

```C++
 void delay_ms(int32_t nms) 
 {
  int32_t temp; 
  SysTick->LOAD = 8000*nms; 
  SysTick->VAL=0X00;//清空计数器 
  SysTick->CTRL=0X01;//使能，减到零是无动作，采用外部时钟源 
  do 
  { 
       temp=SysTick->CTRL;//读取当前倒计数值 
  }
     while((temp&0x01)&&(!(temp&(1<<16))));//等待时间到达 
   
     SysTick->CTRL=0x00; //关闭计数器 
     SysTick->VAL =0X00; //清空计数器 
 }
```

---

```C++
void Delay_us(int16_t nus) 
{
  int32_t temp; 
  SysTick->LOAD = nus*9; //72MHz
  SysTick->VAL=0X00;
  SysTick->CTRL=0X01;
  do 
  { 
    temp=SysTick->CTRL;
  }
  while((temp&0x01)&&(!(temp&(1<<16))));
   
  SysTick->CTRL=0x00; 
  SysTick->VAL =0X00; 
}
```

#### 利用HAL_Delay

[STM32 HAL库学习 常使用的几种延时方式](https://blog.csdn.net/qq_34752070/article/details/82620374)

```C++
// HAL_RCC_GetHCLKFreq()/1000 1ms中断一次，即HAL_Delay函数延时基准为1ms
// HAL_RCC_GetHCLKFreq()/100000  10us中断一次，即HAL_Delay函数延时基准为10us
// HAL_RCC_GetHCLKFreq()/1000000 1us中断一次，即HAL_Delay函数延时基准为1us
HAL_SYSTICK_Config(HAL_RCC_GetHCLKFreq()/1000000);  // 配置并启动系统滴答定时器
```

#### 利用TIM

[HAL库中同时实现微秒级us以及毫秒级ms延时](https://blog.csdn.net/qq_29506411/article/details/109070558)

tim6，中断不使能

```C++
//72M  PSC=71

void Delay_us(uint16_t us) //注意us变量的上限是65535
{
// uint16_t counter= us & 0xffff;
 
  HAL_TIM_Base_Start(&htim6);
  __HAL_TIM_SetCounter(&htim6,0);       // 对上次延时产生的计数清零
 
  us = (us > 4)?(us-2):1;    // 对counter的改变是为了让短时长的延时更精确（通过示波器校正过，timer的时钟是72M）
 
  while( us > __HAL_TIM_GetCounter(&htim6) ) {};
 
  HAL_TIM_Base_Stop(&htim6);
}
```

## 串口

### 串口重定向

[STM32 HAL串口接收常用的几种方式](https://blog.csdn.net/morixinguan/article/details/103474643)
[Stm32 HAL库 USART(发送+接收)全部采用DMA形式](https://blog.csdn.net/xinghunlove123/article/details/89503218)

### [UART成帧]

[UART成帧](https://blog.csdn.net/WANGYONGZIXUE/article/details/121375351)

ESP8266 判断 UART 传来的数据时间间隔，若时间间隔大于 20ms， 则认为一帧结束；否则， 一直接收数据到上限值 2KB， 认为一帧结束。 ESP8266 模块判断UART 来的数据一帧结束后， 通过 WIFI 接口将数据转发出去。
成帧时间间隔为 20ms， 一帧上限值为 2KB。

## SPI

[STM32（HAL）——SPI通信](https://blog.csdn.net/weixin_41082463/article/details/104952605)
[HAL库的学习 —— SPI配置和使用 发送16位和8位数据](https://blog.csdn.net/wanruiou/article/details/97236750)

| Mode                 | 含义         |
| -------------------- | ------------ |
| Full-Duplex Master   | 全双工主模式 |
| Full-Duplex Slave    | 全双工从模式 |
| Half-Duplex Master   | 半双工主模式 |
| Half-Duplex Slave    | 半双工从模式 |
| Receive Only Master  | 仅接收主模式 |
| Receive Only Slave   | 仅接收从模式 |
| Transmit Only Master | 仅发送主模式 |
| Transmit Only Slave  | 仅发送从模式 |

| 参数            | 含义                                           |
| --------------- | ---------------------------------------------- |
| Frame Format    | 框架格式，有Motorola和TI两种                   |
| Data Size       | 数据长度，8bit和16bit两种                      |
| First Bit       | 对齐形式，高位先行和低位先行                   |
| Prescaler       | 预分频，用于控制波特率，波特率=16MHz/Prescaler |
| Clock Polarity  | CPOL，前面有讲                                 |
| Clock Phase     | CPHA，前面有讲                                 |
| CRC Calculation | 是否启用CRC                                    |
| NSS Signal Type | 片选形式，硬件实现还是软件实现                 |

## ADC && DAC

**测量前用 `HAL_ADCEx_Calibration_Start();`校准!**
[【STM32】ADC的基本原理、寄存器（超基础、详细版）](https://blog.csdn.net/qq_38410730/article/details/80071349)

[【STM32】HAL库 STM32CubeMX教程九—ADC](https://blog.csdn.net/as480133937/article/details/99627062)

[STM32—ADC详解](https://blog.csdn.net/qq_43743762/article/details/100067558)

[STM32 HAL库学习系列第1篇 ADC配置 及 DAC配置](https://blog.csdn.net/super828/article/details/79600395)

[STM32使用HAL库实现ADC单通道转换(中断和非中断都有代码)](https://www.cnblogs.com/xingboy/p/10018749.html)

[用STM32内置的ADC实现数字示波器](https://wenku.baidu.com/view/7f69e2c081c758f5f61f67e2.html)

[Cube生成定时器2触发双ADC同步采集并用DMA传输](https://blog.csdn.net/qq_38294949/article/details/106036394)

[STM32H743 ADC1+DMA1 ADC3+BDMA CubeMX配置使用](https://blog.csdn.net/weifengdq/article/details/121802176)

> 要知道，转换后的数据是一个12位的二进制数，我们需要把这个二进制数代表的模拟量（电压）用数字表示出来。比如测量的电压范围是0~3.3V，转换后的二进制数是x，因为12位ADC在转换时将电压的范围大小（也就是3.3）分为4096（2^12）份，所以转换后的二进制数x代表的真实电压的计算方法就是：
>
> y=3.3* x / 4096

## 血泪教训

&ensp&ensp如果用定时器触发adc，无论怎样设置，最高就500khz，但是一旦选择连续采样模式，让adc自动一次接一次采样，由于我配置的adc 14mhz，14个周期，因此自然而然的就是1m的采样频率。
而使用定时器触发时，需要关闭连续转换模式，否则无法通过定时器控制adc的采样频率
然后又不知道咋回事，除了使能连续转换模式，定时器触发永远是500khz的上限，我暂时没有找到不使能连续转换模式的同时使adc速度大于500khz的方法

# 模块

## AD9910

 [AD9910模块数据手册、使用方法详解](https://blog.csdn.net/fraay/article/details/108687441)

## mpu6050

[stm32f103与mpu6050通信详解](https://blog.csdn.net/zj490044512/article/details/83745684)

## DHT11

[HAL库 DHT11 驱动](https://blog.csdn.net/dingyc_ee/article/details/103530982)

## SSD1306 0.96OLED

### 驱动

* [stm32libs 5年前的库 基于stm32f103](https://github.com/SL-RU/stm32libs)
* [stm32-ssd1306](https://github.com/afiskon/stm32-ssd1306)
  > STM32 library for working with OLEDs based on SSD1306, supports I2C and 4-wire SPI. It also works with SH1106, SH1107 and SSD1309 which are compatible with SSD1306.
  > Please see ssd1306/ssd1306_conf_template.h and examples directory for more details.
  >
* [ssd1306-stm32HAL](https://github.com/4ilo/ssd1306-stm32HAL)
  > ssd1306 library for stm32 using stm32-hal library’s. This library works with i2c and is configured for 128x64 oled panels by default.
  > If you search 4-wire SPI support, you can find it in the afiskon/stm32-ssd1306 fork.
  >

### 获取OLED的通信地址

参考[关于Arduino&amp;SSD1306OLED（IIC）显示的学习](https://blog.csdn.net/qq_42860728/article/details/84310160)

> [![](https://shjdgwj.github.io/e6640d881469/images/loading.gif)](https://shjdgwj.github.io/e6640d881469/4.png)
> 模块背面的IIC ADRESSSELECT表示该模块在IIC通信作为从机时的地址，当中间的脚用电阻和左边接起来时，地址为0x78，当和右边接起来时，地址为0x7A。
>
> 图片所示的通信地址是0x78

### 显示与取模

[关于0.96OLED的显示过程详解（I2C通信方式）](https://blog.csdn.net/u010858987/article/details/103362144)

## ESP8266

[【stm32】wifi ESP8266的AT指令](https://blog.csdn.net/zDavid_2018/article/details/108349593)

* `__HAL_UART_ENABLE_IT(&huart1, UART_IT_RXNE);`中断模式下使用此函数使能中断

[arduino+ESP8266模块使用AT指令设置wifi](https://dsx2016.com/?p=1510)

```C++
void USART1_IRQHandler(void)
{
  /* USER CODE BEGIN USART1_IRQn 0 */
  uint8_t ch = 1;
  if (__HAL_UART_GET_FLAG(&huart1, UART_FLAG_RXNE) != RESET)
  {
    //读寄存器
    ch = (uint16_t)READ_REG(huart1.Instance->DR);

    //将串口1的数据 写入串口3（串口3将数据 -> esp8266）
    WRITE_REG(huart3.Instance->DR, ch);
  }
  /* USER CODE END USART1_IRQn 0 */
  HAL_UART_IRQHandler(&huart1);
  /* USER CODE BEGIN USART1_IRQn 1 */

  /* USER CODE END USART1_IRQn 1 */
}
void USART3_IRQHandler(void)
{
  /* USER CODE BEGIN USART3_IRQn 0 */
  uint8_t ch = 1;
  if (__HAL_UART_GET_FLAG(&huart3, UART_FLAG_RXNE) != RESET)
  {
    ch = (uint16_t)READ_REG(huart3.Instance->DR);

    //给串口1
    WRITE_REG(huart1.Instance->DR, ch);
  }

  /* USER CODE END USART3_IRQn 0 */
  HAL_UART_IRQHandler(&huart3);
  /* USER CODE BEGIN USART3_IRQn 1 */

  /* USER CODE END USART3_IRQn 1 */
}
```

# 算法

## FFT

1. [matlab中fft运算后需要对幅值乘2除N](https://blog.csdn.net/u013028442/article/details/88836515)
2. 矩形窗(Rectangular)：加矩形窗等于不加窗，因为在截取时域信号时本身就是采用矩形截取，所以矩形窗适用于瞬态变化的信号，只要采集的时间足够长，信号宽度基本可以覆盖整个有效的瞬态部分。
   汉宁窗(Von Hann)：如果测试信号有多个频率分量，频谱表现的十分复杂，且测试的目的更多关注频率点而非能量的大小。在这种情况下，需要选择一个主瓣够窄的窗函数，汉宁窗是一个很好的选择。
   flattop窗：如果测试的目的更多的关注某周期信号频率点的能量值，比如，更关心其EUpeak,EUpeak-peak,EUrms，那么其幅度的准确性则更加的重要，可以选择一个主瓣稍宽的窗，flattop窗在这样的情况下经常被使用。
3. [【Get深一度】矩形窗/bartlett/Blackman/hamming/Hanning/kaiser -相控阵雷达原理](https://blog.csdn.net/u013346007/article/details/54142981)

# 问题

## STM32CUBEMX BUG

1. 版本6.5.0 stm32h743 配置ADC时无法配置其时钟。
   版本6.3.0解决此问题
2. STM32 DMA初始化代码要在ADC前面。
3. [STM32CubeMX生成代码时防止UTF-8乱码](https://blog.csdn.net/weixin_49497012/article/details/118499056)
   **添加环境变量**
   * 变量名称：JAVA_TOOL_OPTIONS
   * 变量值：-Dfile.encoding=UTF-8

## DMA循环模式导致hal_delay失效

[可能的原因](https://blog.csdn.net/apple_2333/article/details/96962574)
心态爆炸！
野火教程

```scss
uint32_t adcData[5];
HAL_ADCEx_MultiModeStart_DMA(&hadc1, (uint32_t *)&adcData, sizeof(adcData));
```

而实际上应该写

```scss
uint32_t adcData[5];
HAL_ADCEx_MultiModeStart_DMA(&hadc1, (uint32_t *)&adcData, 5);
//HAL_ADC_Start_DMA(&hadc1, (uint32_t *)&adcData, 5)
```

`HAL_StatusTypeDef HAL_ADCEx_MultiModeStart_DMA(ADC_HandleTypeDef* hadc, uint32_t* pData, uint32_t Length)`
`HAL_StatusTypeDef HAL_ADC_Start_DMA(ADC_HandleTypeDef* hadc, uint32_t* pData, uint32_t Length)`
Length的参数不应该是数据长度，应该是数据数量。
[![](https://shjdgwj.github.io/e6640d881469/images/loading.gif)](https://shjdgwj.github.io/e6640d881469/5.png)
在此问题耗时6小时，特此纪念

注/HAL_ADC_Start_DMA()的data变量为uint32_t的原因是hal为了程序方便移植，虽然f1只支持12位精度的adc，但是其他芯片支持更高精度的adc。

## hal 根据exti获取触发中断的管脚

比如stm32f103c8t6 PA15和PC15都是EXTI15 那触发时我怎么知道是哪个脚触发？

> [https://zhidao.baidu.com/question/1642129457849988940.html](https://zhidao.baidu.com/question/1642129457849988940.html)
> 比如，来自PA0的外部中断，可以通过库函数
> (EXTI_GetITStatus(EXTI_Line0)!=RESET);
> 判断外部中断来源是不是来源于端口0（至于是PA0还是PB0可通过查询中断来源进行判断，但不建议这么用，所以设置外部中断的端口建议不要重复，比如使用了PA0，就不要使用PB0之类的）
> 另外要注意：端口0-4有自己独立的外部中断函数入口，5-9和10-15两组分别共用两个外部中断函数入口

## HAL_GPIO_EXTI_Callback

HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin) 里面一调用函数就锁死，直接写语句没问题

> [https://www.stm32cube.com/question/615](https://www.stm32cube.com/question/615)
> HAL_InitTick(uint32_t TickPriority)这个函数要重新定义下，把systick的主优先级定义为0x0000(最高），之前HAL库给出的宏参数TICK_INT_PRIORITY 0X000f(最低）。导致一进中断
> HAL的时钟就停摆了。

## "Insufficient RAM for Flash Algorithms"出错原因及解决方案

Insufficient RAM for Flash Algorithms”错误一般会有一个“cannot load flash programming algorithm !”的提示窗口
[![](https://shjdgwj.github.io/e6640d881469/images/loading.gif)](https://shjdgwj.github.io/e6640d881469/6.png)
如图更改为 `0x0000FFF4`
[![](https://shjdgwj.github.io/e6640d881469/images/loading.gif)](https://shjdgwj.github.io/e6640d881469/7.png)

注：C++ include 添加 `Drivers\CMSIS\DSP\Include`

## stm32函数中大数组问题

> 在以stm32构建系统的时候，当用户自己编写函数时，发现函数出现意想不到的结果，其中一项你需要注意的是看你的函数中有没有大的数组，或者说查看你函数中临时变量的总量是不是超过了系统设置的堆栈的最大值这类问题编译器是不会给出错误的，相应的当出现程序不能给出想要的结果的时候，我们需要特别注意这类问题。对系统设置函栈最大值的宏一般放在系统的启动文件中，具体的是startup_stm32xxxx.s这个启动文件的Stack_Size这个宏

## STM32H743 STM32CUBEMX6.3.0 ADC时钟为什么能配置为80MHz？

[Clock source for ADC of The STM32H7 CPU](https://community.st.com/s/question/0D53W000013pKvlSAE/clock-source-for-adc-of-the-stm32h7-cpu)

ADC 时钟配置成160MHz，12位可以达到12M采样率（ADC直连通道） 14位5M采样率

[![](https://shjdgwj.github.io/e6640d881469/images/loading.gif "image-20220721150437943")](https://shjdgwj.github.io/e6640d881469/image-20220721150437943.png)## STM32F429 HAL 定时器触发DMA 内存到内存

[https://blog.csdn.net/qq_38974298/article/details/118158575](https://blog.csdn.net/qq_38974298/article/details/118158575)

[http://www.efton.sk/STM32/bt.c](http://www.efton.sk/STM32/bt.c)

## [Keil MDK下如何设置非零初始化变量 - 基于Arm Compiler 6](https://blog.csdn.net/zhzht19861011/article/details/124904070)

| Arm Compiler 5 属性                           | Arm Compiler 6 属性                                   | 描述                                                                                                                            |
| --------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `__attribute__((at(address)))`              | `__attribute__((section(".ARM.__at_address")))`     | Arm Compiler 6 中的 armlink 仍然支持以 `.ARM.__at_address` 的形式放置段                                                       |
| `__attribute__((at(address), zero_init))`   | `__attribute__((section(".bss.ARM.__at_address")))` | Arm Compiler 6 中的 armlink 支持以 `.bss.ARM.__at_address` 的形式放置零初始化段。 `.bss` 前缀区分大小写，并且必须全部小写。 |
| `__attribute__((section(name), zero_init))` | `__attribute__((section(".bss.name")))`             | `name` 是你选择的名字。 `.bss` 前缀区分大小写，并且必须全部小写。                                                           |
| `__attribute__((zero_init))`                | 不支持 默认将零初始化变量放在 `.bss` 段。           | 如果变量具有初始值设定项，则 Arm Compiler 5 会生成错误。 否则，它将零初始化变量放在 `.bss`段。                                |

> ### 5. Arm® Compiler 5 如何防止未初始化变量被初始化为0
>
> 1. 定义变量时，使用编译器扩展属性 `__attribute__((section("name"), zero_init))`来将变量放入指定段中。其中 `section("name")`选择一个指定的段，`zero_init`告诉编译器将变量放入ZI段。
>
>    ```c
>    uint32_t phy_link_init_flag __attribute__((section("NO_INIT"), zero_init));
>    1
>    ```
> 2. 在分散加载文件中，定义名为 `NO_INIT`的段。注意该段所在的可执行域要具有 `UNINIT`属性。
>
>    ```c
>    LR_IROM1 0x00000000 0x00080000  {    ; load region size_region 从0扇区开始
>      ER_IROM1 0x00000000 0x00080000  {  ; load address = execution address 
>       *.o (RESET, +First)
>       *(InRoot$$Sections)
>       .ANY (+RO)
>      }
>      RW_IRAM1 0x10000000 0x0000F000  {  ; RW data
>       .ANY (+RW +ZI)
>      }
>
>      RW_IRAM2 0x1000F000 UNINIT 0x00001000  {
>       .ANY (NO_INIT)
>      }
>    }
>    1234567891011121314
>    ```
>
> ### 6. Arm® Compiler 6 如何防止未初始化变量被初始化为0
>
> 1. 定义变量时，使用编译器扩展属性 `__attribute__((section("name")))`来将变量放入指定段中。其中 `section("name")`选择一个指定的段。
>
>    ```c
>    uint32_t phy_link_init_flag __attribute__((section(".bss.NO_INIT")));
>    1
>    ```
> 2. 在分散加载文件中，定义名为 `.bss.NO_INIT`的段，其中前缀 `.bss`是必须的，并且只能为小写。这个前缀表明该数据段具有ZI属性。注意该段所在的可执行域要具有 `UNINIT`属性
>
>    ```c
>    LR_IROM1 0x00000000 0x00080000  {    ; load region size_region 从0扇区开始
>      ER_IROM1 0x00000000 0x00080000  {  ; load address = execution address 
>       *.o (RESET, +First)
>       *(InRoot$$Sections)
>       .ANY (+RO)
>      }
>      RW_IRAM1 0x10000000 0x0000F000  {  ; RW data
>       .ANY (+RW +ZI)
>      }
>
>      RW_IRAM2 0x1000F000 UNINIT 0x00001000  {
>       .ANY (.bss.NO_INIT)
>      }
>    }
>    ```

## 定时器触发DMA控制GPIO

[Parallel transmission using GPIO and DMA (like AN4666)](https://community.st.com/s/question/0D53W00000Eo9rLSAR/parallel-transmission-using-gpio-and-dma-like-an4666)

[stm32h7-gpio-toggling](https://metebalci.com/blog/stm32h7-gpio-toggling/)

# 参考博客

[STM32学习笔记](https://shjdgwj.github.io/e6640d881469/)
[Yngz_Miao《嵌入式》STM32开发笔记](https://blog.csdn.net/qq_38410730/category_7511110.html)
[Z小旋 STM32](https://blog.csdn.net/as480133937/category_9188655.html)
[mculover666](http://www.mculover666.cn/)
