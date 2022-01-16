---
layout: blog
title: STMCUBEMX 配置指北
date: 2022-01-12 16:43:35
tags: 实验记录
---

> 希望每做一步都有因可依，有据可循。

## revision history 

### 第三版电路:

- 对每相电机驱动芯片，每个霍尔传感器对应了一个中断。
- 使用ADC测量$V_{SENSE}$
- 使用SPI与ICM-42688-P 进行通讯
- 单点接地
- [ ] BOOT0 悬空
- [ ] PA11 无用？

#### GPIO 配置

| Pin_Port | Type   | Label    | Specific                                                     |
| -------- | ------ | -------- | ------------------------------------------------------------ |
| PA0      | EXTI   | HALL2C   | Pull-down                                                    |
| PA1      | EXTI   | HALL2B   | Pull-down                                                    |
| PA9      | Output | EN1      | LOW logic level switches OFF all power MOSFETS<br />If not used, it has to be connected to +5V.<br />pull-down |
| PA10     | Output | FWD/REV1 |                                                              |
| PB11     | EXTI   | HALL1A   | Pull-down                                                    |
| PB12     | EXTI   | HALL1B   | Pull-down                                                    |
| PB13     | EXTI   | HALL1C   | Pull-down                                                    |
| PC0      | Output | EN2      | LOW logic level switches OFF all power MOSFETS<br />If not used, it has to be connected to +5V.<br />pull-down |
| PC1      | Output | FWD/REV2 |                                                              |
| PC3      | EXTI   | HALL2A   | Pull-down                                                    |
| PC6      | Output | Brake1   | LOW logic level switches ON all high side power MOSFETs,<br/>implementing the brake function.<br/>If not used, it has to be connected to +5 V. |
| PF0      | Output | Brake2   | LOW logic level switches ON all high side power MOSFETs,<br/>implementing the brake function.<br/>If not used, it has to be connected to +5 V. |

<u>TIPS: FWD/REV1 and FWD/REV2 have to be set in a reversed way.</u>

#### Analog 配置

| Analog Device | Configuration                                           | Pin_Port | purpose      |
| ------------- | ------------------------------------------------------- | -------- | ------------ |
| ADC1          | IN15 Single-ended                                       | PB0      | $V_{SENSE1}$ |
| ADC2          | IN15 Single-ended                                       | PB15     | $V_{SENSE2}$ |
| OPAMP1        | PGA Connected-INVERTINGINPUT_IO0_IO1_BIAS-DAC3_OUT1-INP | PA2      | OPAMP1_VOUT  |
|               |                                                         | PA3      | OPAMP1_VINM0 |
|               |                                                         | PC5      | OPAMP1_VINM1 |
| OPAMP3        | PGA Connected-INVERTINGINPUT_IO0_IO1_BIAS-DAC3_OUT2-INP | PB1      | OPAMP3_VOUT  |
|               |                                                         | PB2      | OPAMP3_VINM0 |
|               |                                                         | PB10     | OPAMP3_VINM1 |
| DAC3          | OUT1 Connected                                          |          |              |
|               | OUT2 Connected                                          |          |              |

## ITM 配置




## middleware 配置

### FreeRTOS

