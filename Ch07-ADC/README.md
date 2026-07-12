# 第7章 ADC

> 芯片型号：STM32F103C8T6 | 系列：STM32F1 | 内核：ARM Cortex-M3 | 主频：72MHz | RAM：20K | ROM：64K | 供电：2.0~3.6V | 封装：LQFP48

## 7-1 AD单通道

## 7-2 AD多通道

### ADC (Analog-Digital Converter) 模拟-数字转换器

- ADC 可将引脚上连续变化的模拟电压转换为内存中存储的数字变量，建立模拟电路到数字电路的桥梁
- 12位逐次逼近型 ADC，1μs 转换时间

**关键参数**：
- 分辨率：12位 AD 值，范围 0 ~ 2^12-1 (0 ~ 4095)
- 转换时间：从 AD 转换开始到产生结果需 1μs 时间 ↔ 频率 1MHz

- 输入电压范围：0 ~ 3.3V，转换结果范围：0 ~ 4095
- 18个输入通道，可测量 16个外部 和 2个内部 信号源
  - 内部温度传感器
  - 内部参考电压 1.2V
- 寄存器只能存一个值，需 DMA 帮忙存
- 规则组和注入组两个转换单元（增强功能）
- 模拟看门狗自动监测输入电压范围
- STM32F103C8T6 ADC 资源：ADC1、ADC2，10个外部输入通道

### 转换模式（4种）

1. **单次转换，非扫描模式**
2. **连续转换，非扫描模式**
3. **单次转换，扫描模式**
4. **连续转换，扫描模式**

### 数据对齐 → 右对齐
| 0 | 0 | 0 | 0 | D11 | D10 | D9 | D8 | D7 | D6 | D5 | D4 | D3 | D2 | D1 | D0 |

### 转换时间

- AD 转换的步骤：采样、保持、量化、编码
- STM32 ADC 的总转换时间：
  - Tconv = 采样时间 + 12.5 个 ADC 周期
  - 例：当 ADCCLK = 14MHz，采样时间为 1.5 个 ADC 周期
  - Tconv = 1.5 + 12.5 = 14 个 ADC 周期 = 1μs

### 校准（ADC 初始化的最后加上）

- `RCC_ADCCLKConfig()`：配置 ADCCLK 分频器（2、4、6、8 分频）
- `ADC_DeInit()`：恢复缺省配置
- `ADC_Init()`：初始化
- `ADC_StructInit()`：结构体初始化
- `ADC_Cmd()`：给 ADC 上电（开关控制）
- `ADC_DMACmd()`：开启 DMA 输出信号
- `ADC_ITConfig()`：控制中断（中断输出控制）

校准流程：
```c
ADC_ResetCalibration();              // 复位校准
ADC_GetResetCalibrationStatus();     // 获取复位校准状态
ADC_StartCalibration();              // 开始校准
ADC_GetCalibrationStatus();          // 获取开始校准状态

ADC_SoftwareStartConvCmd()：软件触发控制（触发控制）
ADC_GetFlagStatus()：获取标志位状态
ADC_DiscModeChannelCountConfig()：每隔几个通道间断一次
ADC_DiscModeCmd()：是否启用间断模式
ADC_RegularChannelConfig()：ADC 规则组通道配置，给序列每个位置填写指定通道
ADC_ExternalTrigConvCmd()：是否允许外部触发转换
ADC_GetConversionValue()：获取 AD 转换的数据寄存器，读取转换结果
ADC_GetDualModeConversionValue()：双 ADC 模式读取转换结果
模拟看门狗：
ADC_AnalogWatchdogCmd();              // 是否启动模拟看门狗
ADC_AnalogWatchdogThresholdsConfig(); // 配置高、低阈值
ADC_AnalogWatchdogSingleChannelConfig(); // 配置看门狗的通道
ADC_TempSensorVrefintCmd();           // 开启内部的两个通道（温度传感器、内部参考电压控制）
ADC 基本结构
GPIO(16) ──┐
温度 ──────┼──→ [AD转换器] ──→ [AD数据寄存器] ──→ 中断输出控制 ──→ NVIC
VREFINT(4)─┘    规则组(x1)      规则组结果 x1
                注入组(x4)      注入组结果 x4
                    ↑                ↑
                START(触发控制)   EOC
                    ↑
                CLOCK(RCC)
                
触发控制：软件 / 硬件（定时器、外部中断引脚）

### 实验项目
- 7-1 AD单通道
- 7-2 AD多通道

### 实验记录
- 📷 [7-2 AD多通道实验GIF](notes/7-2 AD多通道实验.gif)

---

[← 返回根目录](../README.md)