# 第8章 DMA

## 8-1 DMA数据转运

### DMA (Direct Memory Access) 直接存储器存取

- DMA 可以提供外设和存储器或存储器和存储器之间的高速数据传输，无需 CPU 干预，节省 CPU 的资源
- 12个独立可配置通道：DMA1（7个），DMA2（5个）
- 每个通道都支持软件触发和特定硬件触发
- STM32F103C8T6 DMA 资源：DMA1（7个）

### DMA 基本结构

    方向
      ↓
外设 ──→ 外设寄存器 ←────→ Flash/SRAM ←── 存储器
│           ↑                ↑            │
│           └────────────────┘            │
│                                         │
起始地址 ───────────────────────────────────┤
数据宽度 ───────────────────────────────────┤
地址是否自增 ───────────────────────────────┘
传输计数器 ←── 自动重装器
↑
└── 触发选择 ──→ 硬件触发 / 软件触发 ──→ 开关控制
↑
M2M

> **注**：没有自动重装且传输计数器等于 0 时，DMA 不会再进行转运。需要 `DMA_Cmd` 给 DISABLE，关闭 DMA，再为传输计数器写一个大于 0 的数，再给 ENABLE，开启 DMA。（必须先关 DMA 再给数）

### DMA 库函数

```c
DMA_DeInit();                // 恢复缺省配置
DMA_Init();                  // 初始化
DMA_StructInit();            // 结构体初始化
DMA_Cmd();                   // 开启 DMA
DMA_ITConfig();              // 中断输出使能
DMA_SetCurrDataCounter();    // 给传输计数器写数据
DMA_GetCurrDataCounter();    // 返回传输计数器的值
DMA_GetFlagStatus();         // 获取标志
DMA_ClearFlag();             // 清除标志
DMA_GetITStatus();           // 获取中断状态
DMA_ClearITPendingBit();     // 清除中断挂起位
```
## 8-2 DMA+AD多通道
### 实验项目
8-1 DMA数据转运
8-2 DMA+AD多通道
### 踩坑记录 ⚠️
```c
8-2 DMA+AD多通道 - 数据不更新
现象：ADC 多通道数据无法正常转运。
原因：DMA_InitStructure.DMA_PeripheralBaseAddr 赋值时漏写了取地址符 &。
// ❌ 错误代码
DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)ADC1->DR;   // 漏了 &

// ✅ 正确代码
DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;  // 必须取地址
解决：ADC1->DR 是结构体里的值，不是指针，必须加 & 取地址。
```
### 实验记录
- 📷 [8-2 DMA+AD多通道实验GIF](notes/8-2 DMA+AD多通道实验.jpg)

---

[← 返回根目录](../README.md)
