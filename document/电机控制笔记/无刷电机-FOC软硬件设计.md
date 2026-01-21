无刷电机-FOC软硬件设计

# 一、硬件

设计制作一款 **无刷电机驱控板**

## 路线

开发环境搭建 -> 硬件准备 -> 技术手册 -> 设计PCB -> 交付生产 -> 购买元器件 -> 焊接元器件 -> 功能测试 

# 二、软件

开发自己的 **FOC电机控制程序**

**ST电机库**

**Simple FOC库**

## 路线

开发环境搭建 -> STM32CubeMX配置主控芯片 -> Keil开发FOC程序 -> 开环控制电机旋转 -> 恒定力矩控制 -> 转速闭环控制 -> 角度(位置)闭环控制 -> 位置随动控制

算法： **FOC控制** 、 **数据滤波** 、 **PID控制** 、 **震动抑制** 、**电机参数观测**

## 交流电机变频驱动

当两个磁场轴线正对着时，磁场之间有相互吸引力，该力是径向的，不产生转矩；

当两个磁场轴线有一定夹角时，磁场之间有相互吸引力，该力既有径向分量，也切向分量，会产生一定的转矩；

当两个磁场轴线垂直时，磁场之间有相互吸引力，该力主要是切向分量，此时产生最大转矩。

**原理**：按照一定频率分别给U、V、W相通电，交流电机即可旋转。n = 60 f / pn (n--电机旋转频率；f--通电频率；pn--磁极对数)

**所需MCU功能外设**：高级定时器

**任务**：输出6路PWM，上下桥互补

```c
/* 配置系统时钟：
 * 1、RCC--HSE：Crystal/Ceramic Resonator (晶体/陶瓷晶振)
 * 2、时钟树：Input frequency (输入频率)--PLL Source (锁相环时钟源配置HSE)--System Clock (系统时钟源配置PLLCLK)--HCLK (高速总线AHB的时钟信号配置为MCU所支持最高频率)
 * 
 * STM32时钟源(5):
 *     HSI：高速内部时钟，RC振荡器，频率为8MHz
 *     HSE：高速外部时钟，可接【外部无源晶振】(Crystal/Ceramic Resonator 晶体/陶瓷晶振)，或者【外部有源晶振】(BYPASS Clock Resource 旁路时钟源)，频率为4MHz~16MHz
 *     LSI：低速内部时钟，RC振荡器，频率为40kHz
 *     LSE：低速外部时钟，石英晶体，频率为32.768kHz
 *     PLL：锁相环倍频输出，其时钟输入源可选择为HSI/2、HSE或者HSE/2，倍频可选择为2~16倍，频率最大不超过MCU所支持最高频率
 *
 * STM32系统时钟(3)：
 *     HCLK：高速总线AHB的时钟
 *     PCLK：低速总线APB的时钟
 *     FCLK：Cortex clock 内核时钟，即MCU主频
 */

/* 配置高级定时器：
 * 1、TIM1或TIM8--Clock Source (时钟源配置Internal Clock 内部时钟)--Channel1至Channel3 (通道1至3配置PWM Generation CHx CHxN 通道x和互补通道x生成PWM 模式)--Prescaler (预分频系数配置)--Counter Period (计数周期、AutoReload自动重装载值配置)--auto-reload preload (自动重装载值预加载配置使能)--Dead Time (死区时间配置)
 * 2、定时器PWM开始
 *     HAL_TIM_PWM_Start(&htimx, TIM_CHANNEL_x);
 *     HAL_TIMEx_PWM_Start(&htimx, TIM_CHANNEL_x);
 * 3、配置TIM捕获比较寄存器的值
 *     #define PWM_Period (htimx.Init.Period + 1)    // PWM计数周期
 *     #define U0 __HAL_TIM_SET_COMPARE(&htimx , TIM_CHANNEL_x , 0);    // Pulse值为0，PWM占空比为0
 *     #define U1 __HAL_TIM_SET_COMPARE(&htimx , TIM_CHANNEL_x , PWM_Period/10);    // Pulse值为PWM计数周期的1/10，PWM占空比为1/10
 * 4、按照一定频率分别给U、V、W相通电 (ms--通电周期)
 *     U1;V0;W0;
 *     HAL_Delay(ms);    // U相通
 *     U0;V1;W0;
 *     HAL_Delay(ms);    // V相通
 *     U0;V0;W1;
 *     HAL_Delay(ms);    // W相通
 */

/* 电机发烫的原因：
 * 共给电机的电流一部分转换为机械能，一部分转换为热能，当电机所需的机械能满足，多余的电流以热能形式消耗，这是开环控制的弊端
 */
```

