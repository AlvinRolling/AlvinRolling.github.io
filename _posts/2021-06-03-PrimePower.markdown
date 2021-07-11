---
title:  "PrimePower 使用笔记"
date:   2021-06-03 15:46:30 +0800
categories: Opt 
toc: true
toc_icon: glasses
---

本文介绍 Synopsys 的功耗分析工具 PrimePower 的使用。

## 历史

PrimePower是PrimeTime的配套工具，用来完成静态和动态的功耗分析。参阅UG历史，可以看到，在2017年以前该工具名为PrimeTiem PX，目前中文网上能够找到的文章也多基于该名称。17年之后更名为PrimePower，进一步升级成为 Synopsys Fusion Compiler 平台中的重要工具。

事实上，再往前看，似乎10年以前PrimePower才是主流，10年后PrimePower被称为过时的工具。课件EDA的发展历史以及厂家命名的变化。（事实存疑）
{: .notice--info}

![DesignFlow]({{site_url}}/assets/PrimePower/DesignFlow.jpg)

从图中看到，相比之前的PrimeTime PX，新增加了对于RTL Power Analysis 的支持。该方法需要RTL Architect 工具的引入，在前端便完成初步的综合和版图信息。在本文中对该内容不做介绍。

## 如何使用

### 何时进行功耗分析

功耗分析需要拿到门级的网表，才能后与SC Lib提供的功耗信息对应。因此，通常在综合或者P&R之后，得到门级网表，以及寄生等造成的延时信息后，才能够做实际的功耗分析。但PrimePower提供了读们RTL反标文件的方式，同样可以进行功耗分析，在损失精确度的情况下加快前期探索素速度。

此时所说的RTL反标为标注翻转率等信息，与RTL Architect相关的 Chapter 17 - RTL Power Analysis是不同的概念。
{. notice--info}

### 依赖的文件

- 门级Verilog网表文件。
- SDF 延时文件 / SPEF 寄生参数文件。
- link Library
- SDC 文件
- 翻转率标注文件 (saif / vcd / fsdb)

由于PrimePower与PrimeTime紧密结合，使用SPEF寄生参数文件，让工具在此时自己更新内部的timing信息。相比DC/ICC写出的sdf文件更加准确，但消耗的时间更长。如果只做功耗分析，可以只用sdf文件，但如果同时做STA，推荐使用 SPEF 文件。

## 翻转标注

通过读入VCS/FSDB波形文件文件进行翻转标注，或者通过`set_switching_activity`进行手动进行翻转率设置。推荐通过读入波形文件的方式，功耗的结果更为准确。

几种不同的仿真方式下，标注的翻转结果及时间开销比较如下图

![AnnotateMethod]({{site_url}}/assets/PrimePower/AnnotateMethod.jpg)

对应的 `read_vcd / read_fsdb` 的参数设置如下。可以看到，只有上方的 full-time gate-level，才能给出准确的 time-based ananlysis 功耗值，其余两种方法给出的都是 cycle-accurate peak power，功耗在一个时钟周期内统计并作图。（见后方，Cycle-Accurate Peak Power Analysis）

![EventFileType]({{site_url}}/assets/PrimePower/EventFileType.jpg)

### Gate-Level 标注

波形文件来自 post-synthesis / post implementation 的门级网表仿真。
如果仿真时没有读入 sdf 反标文件，则 `read_vcd / read_fsdb` 需指定 `-zero_delay` 参数。如果带 sdf 延时信息，则无需指定参数，可以给出准确的功耗和时间关系波形。

### RTL 标注

用于设计早期，减少post-synthesis simulation的时间。通过RTL仿真的波形结果，来估计门级网表的功耗。为此，需要使用 `set_rtl_to_game_name`命令，来指定门级网表和RTL之间的 name mapping。实际流程中，在DC阶段，可以使用下方命令自动生成name mapping文件，之后在 PrimePower 中 source 生成的文件即可。

```tcl
saif_map -type primepower -write_map output.namemap
# -type supportes
#       * name_map (the default)
#       * ptpx
#       * primepower      
```

从功能上来理解，该方法得到的功耗结果应该与DC给出的结果类似，都是从RTL网表得到结果，且都没有考虑延时信息。

## Time-Based Analysis

- 多线程 - Native Distributed Peak Power Analysis. (Chapter 7)

```tcl
set power_enable_concurrent_event_analysis true
```

- Concurrent Power Analysis for Discrete Time Intervals

```tcl
read_fsdb –time {A B C D} 
```

performs power analysis over intervals (A-B) and (C-D).

- Enable the concurrent power analysis.
• Set max_process greater than or equal to the number of the discrete intervals mentioned with the –time option.
• Set max_cores greater than or equal to the number of the discrete intervals mentioned with the –time option.
• PrimePower accepts the discrete time intervals with the –timeoption and uses the discrete intervals for concurrent power analysis.
• For example, in the concurrent mode, read_fsdb –time {A B C D} splits the time window as two intervals (A-B) and (C-D). Then, it runs the concurrent power analysis for (A-B) and (C-D) intervals.

TODO: To be continued.

## Viewing Power Waveforms

使用下面命令指定输出功耗波形文件格式，支持 FSDB (.fsdb) 和 Synopsys .out (ASCII format)波形输出

```tcl
set_power_analysis_options –waveform_format [fsdb|out|none]
```

不同类型波形查看器支持的波形格式如下，推荐使用 Verdi 自带的 nWave 打开。

| Waveform Viewers | FSDB | Out |
|------------------|------|-----|
| nWave | Supported | Supported |
| Custom WaveView | Supported | Supported |
| CosmoScope | Supported | Not Supported |

## Analyzing Hierarchical Designs

To speed up the peak power analysis process, PrimePower performs distributed peak-power analysis using the distributed multi-scenario analysis (DMSA) infrastructure available in the PrimeTime tool. The DMSA flow evaluates several scenarios in parallel by using multiple processors. This mechanism provides faster turnaround time and the ability to analyze the results from the multiple scenarios in parallel. For timing analysis, a scenario is a set of operating conditions and modes; for power analysis, a scenario is a time window in the logic simulation activity file.

TODO: To be continued.

## FAQ

**Q1:** read_vcd / read_fsdb 的 `-zero_delay` 参数有什么作用

**A1:** 见“翻转标注”一章。 `-zero_delay` 表明读入的波形文件没有延时信息。

**Q2:** 功耗分析时的延时信息来自哪里？波形文件还是反标文件？

**A2:** 从PrimeTime UG中可以看到，反标文件 (sdf / spef) 仅用来做timing analysis。
> Back-annotation is the process of reading delay or parasitic resistance and capacitance values from an external file into the tool for timing analysis.

而在PrimePower UG中，并没有体现反标的内容。从上一个问题中可以推断出，功耗分析时候的延时信息来自波形文件。如果生成波形文件时，是zero delay模式（即仿真时没有读入反标 sdf），则无法进行准确的 timinng based analysis.

**Q3:** *WARN* This FSDB file has higher version(5.8) than the current FSDB Reader(5.7), so that it does not know how to read it.

**A3:** PrimePower 读取 fsdb 文件的时候，会先调用 `fsdb2vcd` 将其转化为VCD文件。该问题的出现是由于 PrimePower **fsdb_reader** 的版本太低。然后实际上 Verdi 内部自己就有 `fsdb2vcd` 工具，我们只需要设置为使用 Verdi 的 `fsdb2vcd` 命令完成波形文件格式转化即可。设置如下的环境变量。参考 [Solvnet](https://solvnetplus.synopsys.com/s/article/Seeing).

```bash
setenv NOVAS_HOME /user/EDA_Tools/Synopsys/verdi/R-2020.12-SP1

setenv NOVAS_VERSION R-2020.12-SP1

setenv LD_LIBRARY_PATH /user/EDA_Tools/Synopsys/verdi/R-2020.12-SP1/share/vcst/linux64/:$LD_LIBRARY_PATH
```

各版本 Verdi 与 FSDB 文件的对应关系如下图：

![FSDB_Version]({{site_url}}/assets/PrimePower/fsdb_version.jpg)

## Cycle-Accurate Peak Power Analysis

TODO:

## 基本例子

```tcl
#################################################################################
# PrimePower Reference Methodology Script
# Script: pwr.tcl
# Version: O-2018.06 (Jul 17, 2018)
# Copyright (C) 2008-2018 Synopsys All rights reserved.
################################################################################




# Please do not modify the sdir variable.
# Doing so may cause script to fail.
set sdir "." 



##################################################################
#    Source common and pwr_setup.tcl File                         #
##################################################################

source $sdir/rm_setup/common_setup.tcl
source $sdir/rm_setup/pwr_setup.tcl


# make REPORTS_DIR
file mkdir $REPORTS_DIR

# make RESULTS_DIR
file mkdir $RESULTS_DIR 



##################################################################
#    Search Path, Library and Operating Condition Section        #
##################################################################

# Under normal circumstances, when executing a script with source, Tcl
# errors (syntax and semantic) cause the execution of the script to terminate.
# Uncomment the following line to set sh_continue_on_error to true to allow 
# processing to continue when errors occur.
#set sh_continue_on_error true 


# Enabling CCS-based waveform propagation. This variable needs to be set before link_design.
# CCS waveform analysis requires libraries that contain CCS timing and CCS noise data. 
# Please make sure the libraries have passed check_library checks in Library Compiler.  
set delay_calc_waveform_analysis_mode  full_design
  
set power_enable_timing_analysis true 
set power_enable_multi_rail_analysis true 
set power_analysis_mode time_based 

set report_default_significant_digits 3 ;
set sh_source_uses_search_path true ;
set search_path ". $search_path" ;


##################################################################
#    Netlist Reading Section                                     #
##################################################################
set link_path "* $link_path"
read_verilog $NETLIST_FILES

current_design $DESIGN_NAME 
link


##################################################################
#    Back Annotation Section                                     #
##################################################################
if { [info exists PARASITIC_PATHS] && [info exists PARASITIC_FILES] } {
foreach para_path $PARASITIC_PATHS para_file $PARASITIC_FILES {
   if {[string compare $para_path $DESIGN_NAME] == 0} {
      read_parasitics $para_file 
   } else {
      read_parasitics -path $para_path $para_file 
   }
}
}



##################################################################
#    Reading Constraints Section                                 #
##################################################################
if  {[info exists CONSTRAINT_FILES]} {
        foreach constraint_file $CONSTRAINT_FILES {
                if {[file extension $constraint_file] eq ".sdc"} {
                        read_sdc -echo $constraint_file
                } else {
                        source -echo $constraint_file
                }
        }
}





##################################################################
#    Setting Derate and CRPR Section                             #
##################################################################


if { [string is double -strict $derate_clock_early_value ] } {
set_timing_derate $derate_clock_early_value -clock -early
}

if { [string is double -strict $derate_clock_late_value ] } {
set_timing_derate $derate_clock_late_value -clock -late
}

if { [string is double -strict $derate_data_early_value ] } {
set_timing_derate $derate_data_early_value -data -early
}

if { [string is double -strict $derate_data_late_value ] } {
set_timing_derate $derate_data_late_value -data -late
}



set timing_remove_clock_reconvergence_pessimism true 
set timing_clock_reconvergence_pessimism same_transition 





##################################################################
#    Update_timing and check_timing Section                      #
##################################################################

update_timing -full
check_timing -verbose > $REPORTS_DIR/${DESIGN_NAME}_check_timing.report


##################################################################
#    Save_Session Section                                        #
##################################################################
save_session ${DESIGN_NAME}_ss


##################################################################
#    Report_timing Section                                       #
##################################################################
report_global_timing > $REPORTS_DIR/${DESIGN_NAME}_report_global_timing.report
report_clock -skew -attribute > $REPORTS_DIR/${DESIGN_NAME}_report_clock.report 
report_analysis_coverage > $REPORTS_DIR/${DESIGN_NAME}_report_analysis_coverage.report
report_timing -slack_lesser_than 0.0 -delay min_max -nosplit -input -net  > $REPORTS_DIR/${DESIGN_NAME}_report_timing.report
report_constraints -all_violators -verbose > $REPORTS_DIR/${DESIGN_NAME}_report_constraints.report
report_design > $REPORTS_DIR/${DESIGN_NAME}_report_design.report
report_net > $REPORTS_DIR/${DESIGN_NAME}_report_net.report


##################################################################  
#    Power Switching Activity Annotation Section                 #  
##################################################################  
read_vcd $ACTIVITY_FILE -strip_path $STRIP_PATH         
report_switching_activity -list_not_annotated           

##################################################################
#    Power Analysis Section                                      #
##################################################################

## set power analysis options                                   
set_power_analysis_options -waveform_format fsdb -waveform_output $REPORTS_DIR/wave

## run power analysis
check_power   > $REPORTS_DIR/${DESIGN_NAME}_check_power.report
update_power  

## report_power
report_power > $REPORTS_DIR/${DESIGN_NAME}_report_power.report


# Clock Gating & Vth Group Reporting Section
report_clock_gate_savings  

# Set Vth attribute for each library if not set already
foreach_in_collection l [get_libs] {
        if {[get_attribute [get_lib $l] default_threshold_voltage_group] == ""} {
                set lname [get_object_name [get_lib $l]]
                set_user_attribute [get_lib $l] default_threshold_voltage_group $lname -class lib
        }
}
report_power -threshold_voltage_group > $REPORTS_DIR/${DESIGN_NAME}_pwr.per_lib_leakage
report_threshold_voltage_group > $REPORTS_DIR/${DESIGN_NAME}_pwr.per_volt_threshold_group

```

功耗报告的结果如下图所示，与DC给出的结果结构相似。注意，功耗的单位默认为W。

```verilog
                    Internal  Switching  Leakage    Total 
Power Group         Power     Power      Power      Power   (     %)   
------------------------------------------------------------------------
io_pad                 0.0000    0.0000    0.0000    0.0000 ( 0.00%) 
memory                 0.0000    0.0000    0.0000    0.0000 ( 0.00%) 
black_box              0.0000    0.0000    0.0000    0.0000 ( 0.00%) 
clock_network       4.688e-04 1.184e-04 5.424e-07 5.878e-04 ( 5.19%)  i 
register            1.522e-03 3.473e-04 5.492e-07 1.870e-03 (16.52%) 
combinational       3.843e-03 4.989e-03 2.637e-05 8.859e-03 (78.28%) 
sequential             0.0000    0.0000    0.0000    0.0000 ( 0.00%) 
  
  Net Switching Power  = 5.455e-03   (48.20%) 
  Cell Internal Power  = 5.834e-03   (51.55%) 
  Cell Leakage Power   = 2.746e-05   ( 0.24%) 
                         --------- 
Total Power            =    0.0113  (100.00%) 
```
