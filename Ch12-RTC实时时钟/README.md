# 第12章 RTC实时时钟

## 12-1 RTC实时时钟

### 时间戳转换

| 函数 | 功能 |
|------|------|
| `struct tm* gmtime(const time_t*);` | 秒计数器转换为日期时间（格林尼治时间） |
| `struct tm* localtime(const time_t*);` | 秒计数器转换为日期时间（当地时间） |
| `time_t mktime(struct tm*);` | 日期时间转换为秒计数器（当地时间） |
| `time_t time(time_t*);` | 获取系统时钟 |

### RTC 基本结构
```c
HSE/128 ──→ 外部高速时钟
LSE    ──→┬→ 多路选择器 ──→ RTCCLK ──→ [PRL 重装寄存器]
LSI    ──→┘                              ↓
内部低速时钟                         [DIV 余数寄存器]
↓
[CNT 32位计数器]
↓
┌─────────────────┬───────────┬──────────┐
↓                 ↓           ↓          ↓
Second          Overflow      Alarm  →  中断输出控制 → NVIC
↑                                          ↑
[ALR 32位闹钟值] ──────────────────────────────┘
```

- **RTCCLK**：RTC 时钟源，可选择 HSE/128、LSE 或 LSI
- **PRL（Prescaler Reload）**：重装寄存器，设置分频值
- **DIV（Divider）**：余数寄存器，递减计数
- **CNT（Counter）**：32位计数器，计数秒数
- **ALR（Alarm）**：32位闹钟值，与 CNT 比较产生闹钟中断
- **Second**：秒中断
- **Overflow**：溢出中断
- **Alarm**：闹钟中断

### BKP 备份寄存器函数

```c
BKP_TamperPinLevelConfig();      // 配置 TAMPER 引脚的有效电平（高触/低触）
BKP_TamperPinCmd();              // 是否开启侵入检测功能
BKP_ITConfig();                  // 是否开启中断
BKP_RTCOutputConfig();           // 可选择在 RTC 引脚上输出时钟信号、RTC 校准时钟、RTC 闹钟脉冲或秒脉冲
BKP_SetRTCCalibrationValue();    // 设置校准值
BKP_WriteBackupRegister();       // 写备份寄存器
BKP_ReadBackupRegister();        // 读备份寄存器
PWR_BackupAccessCmd();           // 备份寄存器访问使能
```
### RCC 时钟配置
```c
RCC_LSEConfig();         // 配置 LSE 外部低速时钟
RCC_LSICmd();            // 配置 LSI 内部低速时钟
RCC_RTCCLKConfig();      // 选择 RTCCLK 时钟源
RCC_RTCCLKCmd();         // 启用 RTCCLK
```

## 12-2 实时时钟

### 实验项目

- 12-1 RTC实时时钟
- 12-2 实时时钟

### 实验记录

- 📷 [12-2 实时时钟实验GIF](notes/12-2 实时时钟实验.gif)

[← 返回根目录](../README.md)