---
title: STM32g431rbt6 随学随记
date: 2021-12-21 17:25:49
tags:
---

### STM32G431RBT6

选择该款芯片主要是因为其内置了运算放大器。

### DAC

Question: 多久可以改变一次数值，响应时间有多少

![image-20211221210455779](STM32g431rbt6-随学随记/image-20211221210455779.png)

操作首先要经过buffer DAC_DORx，什么是hardware trigger 以及哪里确定$t_{SETTLING}$ 

![image-20211221210826642](STM32g431rbt6-随学随记/image-20211221210826642.png)

![image-20211221211614690](STM32g431rbt6-随学随记/image-20211221211614690.png)

什么是DAC output buffer？

![image-20211221213107123](STM32g431rbt6-随学随记/image-20211221213107123.png)

