# 第10章 I2C通信

## 10-1 I2C通信协议

### I2C 通信

- I2C 总线（Inter IC BUS）是 Philips 公司开发的一种通用数据总线
- 两根通信线：SCL（Serial Clock）串行时钟线、SDA（Serial Data）串行数据线
- 同步，半双工
- 带数据应答
- 支持总线挂载多设备（一主多从、多主多从）

### 硬件电路

1. 所有 I2C 设备的 SCL 连一起，SDA 连一起
2. 设备的 SCL 和 SDA 均需配置成开漏输出模式
3. SCL 和 SDA 各添加一个上拉电阻，阻值一般为 4.7kΩ 左右

### I2C 时序基本单元（6个）

#### 起始条件
#### 终止条件

#### 发送一个字节

SCL 低电平期间，主机将数据位依次放到 SDA 线上（高位先行），然后释放 SCL，从机将在 SCL 高电平期间读取数据位，所以 SCL 高电平期间 SDA 不允许有数据变化，依次循环上述过程 8 次，即可发送一个字节。

#### 接收一个字节

SCL 低电平期间，从机将数据位依次放在 SDA 线上（高位先行），然后释放 SCL，主机将在 SCL 高电平期间读取数据位，所以 SCL 高电平期间 SDA 不允许有数据变化，依次循环上述过程 8 次，即可接收一个字节（主机在接收之前，需要释放 SDA）。

#### 总结

| | 低电平 | 高电平 |
|--|--------|--------|
| **发送** | 主机放数据 | 从机读数据 |
| **接收** | 从机放数据 | 主机读数据 |

#### 发送应答：主机在接收完一个字节之后，在下一个时钟发送一位数据，数据 0 表示应答，数据 1 表示非应答。
#### 接收应答：主机在发送完一个字节之后，在下一个时钟接收一位数据，判断从机是否应答，数据 0 表示应答，数据 1 表示非应答（主机在接收之前，需要释放 SDA）。

### I2C 时序

- **指定地址写**：对于指定设备，在指定地址下，写入指定数据
- **当前地址读**：对于指定设备，在当前地址指针指示的地址下，读取从机数据
- **指定地址读**：对于指定设备，在指定地址下，读取从机数据

---

## 10-2 I2C外设

### I2C 外设简介

- STM32 内部集成了硬件 I2C 外设，支持多主机模型，支持 7 位/10 位地址模式
- 支持不同的通讯速度：标准模式（高达 100 kHz）、快速模式（高达 400 kHz）
- 支持 DMA、支持 SMBus 协议
- STM32F103C8T6 I2C 资源：I2C1、I2C2

### I2C 框图

- **数据控制**：数据寄存器 DR、移位寄存器
- **时钟控制**：产生 SCL 时钟
- **SDA 控制**：控制 SDA 线的数据收发
- **整体控制逻辑**：控制整个 I2C 外设的工作模式

### 主机发送时序（7位地址）
起始 → 发送从机地址(写) → 接收应答 → 发送数据1 → 接收应答 → ... → 发送数据N → 接收应答 → 终止

### 软件/硬件波形对比

| 特性 | 软件 I2C | 硬件 I2C |
|------|----------|----------|
| 引脚 | 任意 GPIO | 固定引脚（PB6/PB7 或 PB10/PB11） |
| 时序 | 程序控制，灵活 | 外设自动生成，稳定 |
| 效率 | 占用 CPU | 不占用 CPU |
| 速度 | 受程序影响 | 标准 100kHz / 快速 400kHz |

### I2C 库函数

```c
I2C_DeInit();              // 恢复缺省配置
I2C_Init();                // 初始化
I2C_StructInit();          // 结构体初始化
I2C_Cmd();                 // 使能 I2C 外设
I2C_ITConfig();            // 中断配置

I2C_StartCmd();            // 产生起始条件（软件模拟用，标准库为 I2C_GenerateSTART）
I2C_GenerateSTART();       // 产生起始条件
I2C_GenerateSTOP();        // 产生终止条件
I2C_AcknowledgeConfig();   // 配置应答使能
I2C_SendData();            // 发送数据（写 DR 寄存器）
I2C_ReceiveData();         // 接收数据（读 DR 寄存器）
I2C_Send7bitAddress();     // 发送 7 位地址
I2C_GetFlagStatus();       // 获取标志位
I2C_ClearFlag();           // 清除标志位
I2C_GetITStatus();         // 获取中断标志位
I2C_ClearITPendingBit();   // 清除中断挂起位

// 状态监控函数（可选）
I2C_CheckEvent();          // 检查指定事件
I2C_GetLastEvent();        // 获取最新事件
```
## 10-3 硬件I2C读写MPU6050
MPU6050 简介
6 轴姿态传感器，集成 3 轴加速度计和 3 轴陀螺仪
内部含 DMP 数字运动处理器（本课程未使用）
I2C 从机地址：0xD0（写）/ 0xD1（读），或 0x68（7位地址）
关键寄存器：
PWR_MGMT_1（0x6B）：电源管理，解除睡眠
SMPLRT_DIV（0x19）：采样率分频
CONFIG（0x1A）：配置寄存器
GYRO_CONFIG（0x1B）：陀螺仪配置
ACCEL_CONFIG（0x1C）：加速度计配置
WHO_AM_I（0x75）：设备 ID，固定返回 0x68
硬件 I2C 代码思路
// 1. 开启 RCC 时钟（I2C 和 GPIO）
// 2. 配置 GPIO 为复用开漏输出
// 3. 配置 I2C 外设参数
// 4. 使能 I2C
// 5. 封装 WriteReg / ReadReg 函数
// 6. 初始化 MPU6050，循环读取数据
## 10-4 软件I2C读写MPU6050
软件 I2C 代码思路
用普通 GPIO 模拟 SCL 和 SDA 时序
自行实现 Start、Stop、SendByte、ReceiveByte、SendAck、ReceiveAck 函数
引脚可任意选择，不受硬件外设引脚限制

## 实验项目
10-1 软件I2C读写MPU6050
10-2 硬件I2C读写MPU6050
### 实验记录
- 📷 [10-2 硬件I2C读写MPU6050JPG](notes/10-2 硬件I2C读写MPU6050.jpg)
### 踩坑记录 ⚠️
10-2 硬件I2C读写MPU6050 — 数据全卡在 +09252
现象：MPU6050 的 ID 读取为 0x24，所有轴数据都死死卡在 +09252，传感器完全没有反应。
原因：MPU6050_ReadReg 函数里，I2C_ReceiveData(I2C2) 的返回值没有存到 Data 变量里，直接返回了一个未初始化的栈变量。垃圾值恰好是 0x24。
// ❌ 错误代码
uint8_t MPU6050_ReadReg(uint8_t RegAddress)
{
    uint8_t Data;           // ← 声明了，但没初始化
    
    // ... 前面一堆 I2C 操作 ...
    
    MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_);
    I2C_ReceiveData(I2C2);  // ← 读了 DR 寄存器，但没赋值给 Data！
    
    I2C_AcknowledgeConfig(I2C2, ENABLE);
    return Data;            // ← 返回了一个未初始化的垃圾值
}
修复方法：把 I2C_ReceiveData(I2C2); 改成 Data = I2C_ReceiveData(I2C2);
// ✅ 正确代码
Data = I2C_ReceiveData(I2C2);  // ← 必须把返回值赋给 Data

---

[← 返回根目录](../README.md)