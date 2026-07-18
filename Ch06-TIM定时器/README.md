# 第6章 TIM定时器

&gt; 芯片型号：STM32F103C8T6 | 系列：STM32F1 | 内核：ARM Cortex-M3 | 主频：72MHz | RAM：20K | ROM：64K | 供电：2.0~3.6V | 封装：LQFP48

## 6-1 TIM定时中断

### 定时器类型

| 类型 | 编号 | 总线 | 功能特点 |
|------|------|------|----------|
| **高级定时器** | TIM1、TIM8 | APB2 | 拥有通用+全部功能，额外有重复计数器、死区生成、互补输出、刹车输入等功能 |
| **通用定时器** | TIM2、3、4、5 | APB1 | 拥有基本+全部功能，额外有内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等 |
| **基本定时器** | TIM6、TIM7 | APB1 | 拥有定时中断、主模式触发 DAC 的功能 |

&gt; STM32F103C8T6 定时器资源：TIM1、2、3、4

### 定时中断基本结构
RCC内部时钟 →
GPIO → ETR(外部时钟) ─┬─ 内部时钟模式 / 外部时钟模式1 / 外部时钟模式2 / 编码器模式
其他定时器 ITRx ─┤
GPIO → TIX(捕获通道)─┘
↓
[时基单元] → [中断输出控制] → NVIC
PSC 预分频器 → CNT 计数器
ARR 自动重装器

### 计数器频率计算
CK_CNT = CK_PSC / (PSC + 1)
计数器溢出频率：
CK_CNT_OV = CK_CNT / (ARR + 1) = CK_PSC / (PSC + 1) / (ARR + 1)
溢出时间 = 1 / 溢出频率

例：72MHz / (PSC+1) / (ARR+1)

### 时基单元初始化函数

```c
TIM_DeInit();              // 恢复缺省配置
TIM_TimeBaseInit();        // 时基单元初始化
TIM_Cmd();                 // 使能计数器（运行控制）
TIM_ITConfig();            // 使能中断输出信号（中断输出控制）
时钟源选择
TIM_InternalClockConfig(TIMx);           // 选择内部时钟
TIM_ITRxExternalClockConfig(TIMx, ...);  // 选择 ITRx 其他定时器
TIM_TIxExternalClockConfig(TIMx, ...);   // 选择 TIX 捕获通道的时钟
TIM_ETRClockMode1Config(TIMx, ...);      // 选择 ETR 通过外部时钟模式1输入的时钟
TIM_ETRClockMode2Config(TIMx, ...);      // 选择 ETR 通过外部时钟模式2输入的时钟
TIM_ETRConfig(TIMx, ...);                // 配置 ETR 引脚的预分频器、极性、滤波器等
定时器参数配置
TIM_PrescalerConfig(TIMx, 预分频值, 写入模式);  // 写入预分频值（影子寄存器/缓冲）
TIM_CounterModeConfig(TIMx, 计数器模式);        // 改变计数器的计数模式
TIM_ARRPreloadConfig(TIMx, ENABLE/DISABLE);     // 自动重装器预装功能配置
TIM_SetCounter(TIMx, 数值);                     // 给计数器写入一个值
TIM_SetAutoreload(TIMx, 数值);                  // 给自动重装器写入一个值
TIM_GetCounter(TIMx);                           // 获取当前计数器的值
TIM_GetPrescaler(TIMx);                         // 获取当前预分频器的值
标志位操作
TIM_GetFlagStatus();       // 获取标志位
TIM_ClearFlag();           // 清除标志位
TIM_GetITStatus();         // 获取中断标志位
TIM_ClearITPendingBit();   // 清除中断标志位
```
定时器中断代码思路
RCC开启时钟 → 选择时基单元的时钟源 → 配置时基单元(TIM_TimeBaseInit)
    ↓
配置输出中断控制(TIM_ITConfig)，允许更新中断输出到 NVIC
    ↓
配置 NVIC (NVIC_Init)，在 NVIC 中打开定时器中断通道，并分配一个优先级
    ↓
运行控制 (TIM_Cmd)
    ↓
中断函数：
    if (TIM_GetITStatus(...) == SET) {
        [用户代码]
        TIM_ClearITPendingBit(...);
    }


## 6-2 定时器定时中断 & 定时器外部时钟

RCC 时钟树
启动内部 8MHz RC
启动外部 4~16MHz OSC → 锁相环倍频(x9) → 72MHz
CSS 时钟安全系统
外部失效时，自动切换内部系统时钟
时钟分配
72MHz → AHB预分频(1) → 72MHz
            ↓
        APB1预分频(2) → 36MHz → PCLK1
            ↓
        如果 APB1 预分频 = 1，则频率不变，否则 x2
            ↓
        TIMxCLK (定时器2~7)

        APB2预分频(1) → 72MHz → PCLK2
            ↓
        TIMxCLK (定时器1、8)

## 6-3 TIM输出比较

基本概念
OC (Output Compare)：输出比较
IC (Input Capture)：输入捕获
CC (Capture/Compare)：输入捕获和输出比较的单元
输出比较功能
输出比较可以通过比较 CNT 计数器与 CCR 捕获/比较寄存器值的关系，来对输出电平进行置1、置0或翻转的操作，用于输出一定频率和占空比的 PWM 波形
每个高级定时器和通用定时器都拥有 4 个输出比较通道
高级定时器的前 3 个通道额外拥有死区生成和互补输出的功能
## 6-4 PWM驱动LED呼吸灯 & PWM驱动舵机 & PWM驱动直流电机
PWM (Pulse Width Modulation) 脉冲宽度调制
PWM 波形是一个数字输出信号，也是由高低电平组成的
在具有惯性的系统中，可以通过对一系列脉冲的宽度进行调制，来等效地获得所需要的模拟参量，常应用于电机控速等领域
PWM 参数
| 参数  | 公式                 | 说明                  |
| --- | ------------------ | ------------------- |
| 频率  | Freq = 1 / Ts      | 决定 PWM 等效出来的模拟电压的大小 |
| 占空比 | Duty = Ton / Ts    | 占空比变化的精细程度          |
| 分辨率 | Reso = 1 / (ARR+1) | 占空比变化的精细程度          |

PWM 基本结构
时基单元: PSC → CNT → ARR
                ↓
            CCR 捕获/比较器
                ↓
    CNT < CCR 时, REF 置有效电平
    CNT ≥ CCR 时, REF 置无效电平
                ↓
        极性选择 → 输出使能 → GPIO
PWM 参数计算
PWM 频率：Freq = CK_PSC / (PSC + 1) / (ARR + 1) = 计数器更新频率
PWM 占空比：Duty = CCR / (ARR + 1)
PWM 分辨率：Reso = 1 / (ARR + 1)
舵机 PWM 信号要求
周期为 20ms，高电平宽度为 0.5ms ~ 2.5ms
1/20ms = 0.05kHz = 50Hz
输出比较库函数
TIM_ForcedOC1Config(TIMx, ...);           // 配置强制输出模式
TIM_OC1PreloadConfig(TIMx, ENABLE/DISABLE);  // 配置 CCR 寄存器的预装功能（影子）
TIM_OC1FastConfig(TIMx, ENABLE/DISABLE);     // 配置快速使能
TIM_ClearOC1Ref(TIMx, ...);               // 清除 REF 信号

TIM_OC1PolarityConfig();                  // 单独设置输出极性
TIM_OC1NPolarityConfig();                 // 单独设置输出 N 极性

TIM_CCxCmd();                             // 单独修改输出使能参数
TIM_CCxNCmd();
TIM_SelectOCxM(TIMx, ...);                // 选择输出比较模式
TIM_SetCompare1(TIMx, 数值);              // 单独更改 CCR 寄存器的值 (1~4)
PWM 模式与参考信号
ref（reference）：参考信号| 模式        | 向上计数                              | 向下计数                              |
| --------- | --------------------------------- | --------------------------------- |
| PWM Mode1 | CNT < CCR, REF有效；CNT ≥ CCR, REF无效 | CNT > CCR, REF无效；CNT ≤ CCR, REF有效 |
| PWM Mode2 | CNT < CCR, REF无效；CNT ≥ CCR, REF有效 | CNT > CCR, REF有效；CNT ≤ CCR, REF无效 |

高级定时器 PWM 输出
TIM_CtrlPWMOutputs(TIMx, ENABLE);  // 仅高级定时器使用
在使用高级定时器输出 PWM 时，需要调用这个函数使能主输出，否则 PWM 将不能正常输出
TIM 引脚属于片上外设，配置 GPIO 模式时用复用推挽输出

## 6-5 TIM输入捕获

基本概念
IC (Input Capture)：输入捕获
输入捕获模式下，当通道输入引脚出现指定电平跳变时，当前 CNT 的值将被锁存到 CCR 中
可用于测量 PWM 波形的频率、占空比、脉冲间隔、电平持续时间等参数
每个高级定时器和通用定时器都拥有 4 个输入捕获通道
可配置为 PWMI 模式，同时测量频率和占空比
可配合主从触发模式，实现硬件全自动测量

## 6-6 输入捕获模式测频率 & PWMI模式测频率占空比

测量频率方法
| 方法   | 公式                     | 适用场景          |
| ---- | ---------------------- | ------------- |
| 测频法  | fx = N / T             | 适合测高频         |
| 测周法  | fx = fc / N (T = N/fc) | 适合测低频         |
| 中界频率 | fm = √(fc/T)           | 测频与测周误差相等的频率点 |

输入捕获基本结构
时基单元 (PSC → CNT → ARR)
PWMI 基本结构
GPIO → 滤波器 → 边沿检测 → TI1FP1 → 分频器 → CCR1
                                    ↓
                              TI1FP2 → 分频器 → CCR2
输入捕获库函数
TIM_ICInit(TIMx, &结构体);           // 初始化输入捕获单元
TIM_PWMIConfig(TIMx, &结构体);       // 可快速配置两个通道为 PWMI 模式
TIM_ICStructInit(&结构体);           // 给输入捕获结构体赋一个初始值

TIM_SelectInputTrigger(TIMx, 触发源);   // 选择输入触发源 TRGI（从模式触发源）
TIM_SelectOutputTrigger(TIMx, ...);     // 选择输出触发源 TRGO（主模式触发源）
TIM_SelectSlaveMode(TIMx, 从模式);      // 选择从模式

TIM_SetIC1Prescaler(TIMx, 分频值);      // 单独配置通道1的分频器
TIM_GetCapture1(TIMx);                  // 读取输入捕获通道1的 CCR 值

## 6-7 TIM编码器接口

编码器接口 (Encoder Interface)
编码器接口可接收增量（正交）编码器的信号，根据编码器旋转产生的正交信号脉冲，自动控制 CNT 自增/自减，从而指示编码器的位置、旋转方向和速度
每个高级定时器和通用定时器都拥有 1 个编码器接口
两个输入引脚借用了输入捕获的通道 1 和通道 2（CH1、CH2）
极性调整
调整极性，也就是把任意引脚反相，就可以反转计数方向
把 A、B 引脚调换也可以
编码器接口库函数
TIM_EncoderInterfaceConfig(TIMx, 编码器模式, 通道1极性, 通道2极性);  // 配置编码器接口

## 6-8 编码器接口测速

## 实验项目
- 6-1 定时器定时中断
- 6-2 定时器外部时钟
- 6-3 PWM驱动LED呼吸灯
- 6-4 PWM驱动舵机
- 6-5 PWM驱动直流电机
- 6-6 输入捕获模式测频率
- 6-7 PWMI模式测频率占空比
- 6-8 编码器接口测速
### 踩坑记录 ⚠️
6-3 PWM驱动LED呼吸灯 - PA1 不亮
现象：通道2的 LED 不亮，只有通道1正常呼吸。
原因：初始化函数写错，用了 TIM_OC1Init（通道1）而不是 TIM_OC2Init（通道2）。
// ❌ 错误代码
TIM_OC1Init(TIM2, &TIM_OCInitStructure);   // 通道1初始化通道2

// ✅ 正确代码
TIM_OC2Init(TIM2, &TIM_OCInitStructure);   // 通道2用 OC2Init
解决：改成 TIM_OC2Init 后，双通道都正常。
### 实验记录
- 📷 [6-3 PWM驱动LED呼吸灯实验GIF](notes/6-3 PWM驱动LED呼吸灯实验.gif)
- 📷 [6-5 PWM驱动直流电机实验GIF](notes/6-5 PWM驱动直流电机实验.gif)
- 📷 [6-8 编码器接口测速实验GIF](notes/6-8 编码器接口测速实验.gif)

---

[← 返回根目录](../README.md)