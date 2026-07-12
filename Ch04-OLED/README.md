# 第4章 OLED调试

> 芯片型号：STM32F103C8T6 | 系列：STM32F1 | 内核：ARM Cortex-M3 | 主频：72MHz | RAM：20K | ROM：64K | 供电：2.0~3.6V | 封装：LQFP48

## 4-1 OLED调试工具

## 4-2 OLED显示屏

### OLED 驱动函数

```c
OLED_Init();                    // 初始化
OLED_Clear();                   // 清屏
OLED_ShowChar(1, 1, 'A');       // 在第1行第1列显示一个字符
OLED_ShowString(1, 3, "HelloWorld!");  // 显示字符串
OLED_ShowNum(2, 1, 12345, 5);   // 显示十进制数字（长度为5）
OLED_ShowHexNum(3, 1, 0xAA55, 4);      // 显示十六进制数字
OLED_ShowSignedNum(2, 7, -66, 2);      // 显示有符号十进制数字
OLED_ShowBinNum(4, 1, 0xAA55, 16);     // 显示二进制数字
调试方式总结
表格
方式	说明
串口调试	通过串口通信，将调试信息发送到电脑端，电脑使用串口助手显示调试信息
显示屏调试	直接将显示屏连接到单片机，将调试信息打印在显示屏上
Keil 调试模式	借助 Keil 的调试模式，可使用单步运行、设置断点、查看寄存器及变量等功能
实验项目
4-1 OLED显示屏
实验记录
- 📷 [4-1 OLED显示屏实验照片JPG](notes/4-1 OLED显示屏.jpg)

---

[← 返回根目录](../README.md)