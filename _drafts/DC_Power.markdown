---
title:  "SystemVerilog 功耗估计"
date:   2021-04-28 15:46:30 +0800
categories: Front-End
toc: true
toc_icon: glasses
---
本文介绍在 DC/ICC 中使用SAIF文件进行功耗估计的方法。

## 功耗分类及计算


## SAIF文件生成

### RTL 仿真

单独写一个initial块，打开 gate level monitoring。使用`$toggle_start()`和`toggle_end()`来确定记录翻转的开始和结束时间。`$toggle_report`的三个参数分别为，生成的saif文件名称，记录翻转率的精度，以及记录翻转率的module。

```verilog
initial begin
    $set_gate_level_monitoring("on","sv");
    $set_toggle_region(adder_tb.i_adder);
    // Choose the time interval we want to monitor
    #10ms
    $toggle_start();
    // Wait for the finish of the simulation interval
    #100ms;
    $toggle_stop();
    $toggle_report("adder.saif",1.0e-9,"adder_tb");
end
```

注意，使用SAIF文件记录功能时，VCS编译时需要加上 `-lca` 选项。
{: .notice--warning}

### Gate Level Netlist 仿真

用于DC综合后（例如ICC阶段）的功耗估计及优化。

首先，生成 gate-level forward SAIF 文件。

```bash
lib2saif -output ./lib2saif.saif path_of_your_technology_library/db/your_db.db
```

在 tb initial 块中读入生成的SAIF文件。

```verilog
initial begin
    // Read in the gate-level forward SAIF file
    $read_lib_saif("../../lib2saif/lib2saif.saif");
    $set_gate_level_monitoring("on","sv");
    $set_toggle_region(adder_tb.i_adder);
    // Choose the time interval we want to monitor
    #10ms
    $toggle_start();
    // Wait for the finish of the simulation interval
    #100ms;
    $toggle_stop();
    $toggle_report("adder.saif",1.0e-9,"adder_tb");
end
```

### SAIF 文件使用

在DC或者ICC中，使用 `read_saif` 命令，读入生成的SAIF文件。如果使用MCMM模式，则需要每个Scenario读入一个SAIF文件，可以使用 `report_saif`命令查看翻转率标记的情况，正常情况下标记百分比能够做到 100%

命令如下：

```tcl
#################################################################################
# Setup SAIF Name Mapping Database
#
# Include an RTL SAIF for better power optimization and analysis.
#
# saif_map should be issued prior to RTL elaboration to create a name mapping
# database for better annotation.
################################################################################
saif_map -start

read_saif -auto_map_names -input ${DESIGN_NAME}.saif -instance < DESIGN_INSTANCE > -verbose

report_saif
```



