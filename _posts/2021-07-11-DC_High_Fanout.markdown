---
title:  "DC High Fanout Nets"
date:   2021-04-28 15:46:30 +0800
categories: Front-End
toc: true
toc_icon: glasses
---

本文介绍 DC 综合阶段 High-Fanout Nets 可能遇到的相关问题

## 问题描述

综合时候可能出现 `TIM-134` Warning 

> Design 'Test' containts 2 high-fanout nets. A fanout number of 1000 will be used for delay calculations involving these nets. (TIM-134)

使用`man`查看相关信息，发现 high-fanout nets的delay在综合时候，使用简化的delay model。该方法特别对于一些没有约束的网络（global reset nets, scan enable nets, and so o）有比较好的作用。

与此相关的，有两个参数。`high_fanout_net_threshold`，默认为1000， 当fanout的值超过该阈值时，net会被认为是high-fanout net，使用简化delay model。 `high_fanout_net_pin_capacitance`，默认为1， high-fanout nets使用该值作为单个fanout的pin capacitance.

问题在于，high-fanout nets的默认load为以上两个参数相乘，为1000，远远超过大部分电路的实际情况。造成功耗估计的巨大偏差。

## 问题实例

`create_clock` 命令建立的时钟，本身被认为是 ideal_network，不计算功耗。但综合时使用 clock_gating，门控时钟之后的时钟网络的功耗会包括在最后的power report中。以`lna_readout`模块为例，由于使用了大量的寄存器（>10000），而这些寄存器被同一个时钟门控，DC使用1000作为门控时钟net的capacitance，导致power report里时钟网络的 switching power > 4mW。

## 解决方法

因此，我们修改 `high_fanout_net_pin_capacitance`为一个合理的值，例如 0.002-0.004.重新综合后，得到 switching power 为 1.46e-02mW.

## Notes

- 如果不使用门控时钟，则clock net本身是ideal_network，power reoprt里功耗为0，不会产生该问题。
- 如果门控时钟net的fanout不大，小于`high_fanout_net_threshold`，也不会产生该问题。
- 在大型设计中，由于整体功耗偏大， 该问题产生的多余功耗的估计，可能因为不明显而不被发现。
