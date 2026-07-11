# 6-3 PWM驱动LED呼吸灯

## 实验目的
学习 TIM 输出比较模式，产生 PWM 波形驱动 LED 实现呼吸灯效果。

## 关键配置
- TIM2_CH1 (PA0) → LED 呼吸灯
- TIM2_CH2 (PA1) → LED 呼吸灯（双通道）
- PWM 频率: 1kHz
- 占空比渐变实现亮度变化

## 踩坑记录 ⚠️

### 问题：PA1 不亮
**现象：** 通道2的 LED 不亮，只有通道1正常呼吸。

**原因：** 初始化函数写错，用了 `TIM_OC1Init`（通道1）而不是 `TIM_OC2Init`（通道2）。
```c
// ❌ 错误代码
TIM_OC1Init(TIM2, &TIM_OCInitStructure);  // 这配置的是通道1！

// ✅ 正确代码
TIM_OC2Init(TIM2, &TIM_OCInitStructure);  // 通道2要用 OC2Init