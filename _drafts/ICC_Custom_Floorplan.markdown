---
title:  "ICC 特定形状 Floorplan"
date:   2021-05-24 15:46:30 +0800
categories: Back-End
toc: true
toc_icon: glasses
---

本文介绍 ICC 进行特定形状的 Floorplan 设计以及PIN脚摆放的方法。

## 需求背景

在混合信号电路中，数字电路部分的Floorplan通常需要配合模拟电路，因此需要实现特定的形状，以及固定于模拟电路位置匹配的PIN脚位置。ICC的图形界面中提供了一些快捷的T型、L型等Floorplan的设计方法，但仍然难以满足定制化设计的需求。本文介绍使用脚本完整 ICC Custom Floorplan 的方法。

## 设计流程

### 确定PIN脚位置

在顶层设计中，先固定模拟电路以及数字电路的接口位置，以便两边可以根据定义提前进行后续设计。在Cadence Virtuoso中可以到处PIN脚位置描述文件，其格式示例如下：

```bash
*********************************************
 Library      : layout_test
 Cell Name    : test_cell
 View Name    : layout 
 Created      : Feb 17 15:58:58 2021
*********************************************

"signal_i<0>" 1125.05   1200.05 "M3"    "pin"
"signal_o<0>" 2500.05   2800.05 "M4"    "pin"
```

我们需要自己编写脚本文件（例如Python），将其转化为 ICC 可以读取的 tcl 文件。

```tcl
# pin_physical_constraints.tcl
set_pin_physical_constraints  \
    -pin_name {signal_i[0]}   \
    -layer    {M3}            \
    -off_edge location        \
    -location {1125.05 1200.05}

set_pin_physical_constraints  \
    -pin_name {signal_o[0]}   \
    -layer    {M4}            \
    -off_edge location        \
    -location {2500.05   2800.05}
```

**WARNING:** ICC中确定PIN位置的location坐标，指的是PIN这个方格的左下角。而Virtuoso一般默认为label所在位置。在脚本文件中需要手动加以校正。
{: .notice--warning}

### ICC 读取 PIN 脚约束

```tcl

set ICC_IN_PIN_PAD_PHYSICAL_CONSTRAINTS_FILE "pin_physical_constraints.tcl"
read_pin_pad_physical_constraints $ICC_IN_PIN_PAD_PHYSICAL_CONSTRAINTS_FILE

set_fp_pin_constraints \
    -block_level \
    -hard_constraints {layer location} 
```

**WARNING:** 必须使用 `set_fp_pin_constraints` 命令，否则及同读入了tcl文件，ICC也不会按照它的设定工作。
{: .notice--warning}

## 特定形状 Floorplan

Synopsys 的Student Guide中推荐使用 ``命令。但该命令建立的floorplan的尺寸和用户设定的值有偏差，会自动偏移到最小height/width的整数倍（猜测）。因此如果要准确的定义 floorplan，则不能用此命令。（ICC Version: ...）
{: .notice--danger}

假设我们需要实现下图所示的floorplan，分两步进行。首先建立一个 boundaray，之后再boundary的基础上建立floorplan 并制定 io2core 的值。这样得到的 floorplan的尺寸和我们设定的才能完全一致。

```tcl

########################################
##                 (50,100)  (100,100)
##                      _______
##                     |       |
##        (0,50)       |       |
##            ---------        |
##           |                 |
##           |                 |
##           |                 |
##            -----------------
##        (0,0)             (100,0)
########################################

create_boundary -poly \
{   {0 0} \
    {100 0} \
    {100 100} \
    {50 100} \
    {0 50} \
}

initialize_rectilinear_block \
    -use_current_boundary \
    -left_io2core 10 \
    -right_io2core 10 \
    -top_io2core 10 \
    -bottom_io2core 10
```

NOTES：ICC 还提供了另外一种方法 。。。，但在ICC运行到该命令时总是崩溃退出。
{: .notice--danger}
