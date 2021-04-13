---
layout: posts
title:  "SiliconSmart - Part0"
date:   2021-03-15 16:06:30 +0800
categories: EDA Front-End
---


## SiliconSmart 介绍
SiliconSmart 是业界领先的特征化(Characterization)和建模(Modeling)工具。其支持的建模类型包括.

* Nonlinear Delay Model (NLDM) for timing analysis 
* Nonlinear Power Model (NLPM) for power analysis
* Composite Current Source Models (CCS) for timing and power
* Cadence Effective Current Source Models (ECSM) for timing and power
* I/O Buffer Information Specifications (IBIS)

可以理解为，SiliconSmart 是一种建立单元库的工具，通过它得到我们在综合及后端实现过程中使用的 `.db` 文件，用于后续工作。

### 数据流程
SiliconSmart的工作流程如下，详细过程请参阅SiliconSmart UserGuide.

![alt]( {{ site.url }}/assets/siliconsmart/ss_workflow.png)

其工作流程主要分为4种模式

* **Recharacterization Flow** — This approach will recharacterize data in an existing Liberty model or add new data to an existing Liberty model.
* **Function-Based Flow** — The function-based approach uses functions to automatically determine configuration.
* **Structure-Based Flow** — A function-based approach that uses functions and automatic vector simulation based on the structure of the circuit.
* **Sequence-Based Flow** — The sequence based approach adds a user-defined input stimulus to specify arcs to be characterized and the sequence to be performed, without using automated methods.

由于我们的工作并非自己建立标准单元库，而是在已有单元库的基础上，尝试通过降低工作电压来降低功耗，因此使用上方的 __Recharacterization Flow__。之后的内容也基于该 Flow 展开。

### 工作准备

1. Vendor 提供的 Liberty `.lib` 文件。作为我们re-char flow的参考。
2. Vendor 提供的标准单元post-layout netlist 网表文件。与前端网表的区别在于，增加了寄生RC。如果 Vendor 提供的 netlist 将所有单元包括在同一文件中，需自己将其分开到单独的netlist，每个文件名与标准单元名称对应。统一放在 `./netlist` 文件夹下。
3. Vendor 提供的 PDK 下的 model 文件。注意，model文件类型与选择的仿真器需一致。例如我们使用 `hspice` 作为仿真器，则使用`${PDK_MODEL_DIR}/hspice/`下的model文件。该文件夹下有多个文件，PDK安装时已经自动配置好，我们使用顶层的 `toplevel.l` 即可。

### 脚本编写

此处仅展示SiliconSmart Re-Char Flow过程用到的脚本。具体的讲解放在之后的博文中。

* __configure.tcl__ - SiliconSmart设置文件

```tcl
create_operating_condition op_cond_new

# set Power Supply Voltage and temperature here
add_opc_supplies op_cond_new COREVDD1 0.8
add_opc_grounds op_cond_new COREGND1 0
set_opc_temperature op_cond_new 25

# set Process here
set_opc_process on_cond_all [subst { { .lib '${PDK_MODEL_DIR}/hspice/toplevel.l' TOP_TT}}]

set_config_opt voltage_name_map {COREVDD1 VDD COREGND1 VSS}

define_parameter default {

    # one active pvt for one confugire.tcl file
    set active_pvts {op_cond_new} 

    # HSPICE
    set simulator hspice 
    set simulator_cmd {hspice <input_deck> -o <listing_file> }

    # HSPICE Options
    set simulator_options {
        "common,hspice: probe=1 runlvl=5 numdgt=7 measdgt=7 acct=1 nopage"
    }

    # Simulation Resolution
    set time_res_high 1e-12
    set time_res_low 1e-10

    # Controls which supplies are measured for power consumption
    set power_meas_supplies {VDD}

    # list of grounds supplies used (required for Functional Recognition)
    set power_meas_grounds {VSS}

    # specifies which multi-rail format to be used in Liberty model: none, v1, or v2
    set liberty_multi_rail_format v2

    set slew_derate_upper_threshold 0.9
    set slew_derate_lower_threshold 0.1

    set liberty_max_transition 2
    set liberty_max_capacitance 0.1
    # set liberty_max_capacitance_mode 1

    set archive_condition_on_success compress
    set archive_condition_on_failure yes

    set constraint_mode independent 
    set smc_constraint_style relative_degradation
    set smc_degrade 0.1
    set path_constraint_mode off

    set model_normalized_driver_waveform 0
    set insert_liberty_default_ndw 0
    set merge_pin_ndw_groups 0

    # LOAD SHARE PARAMETERS
    #  job_scheduler: 'lsf' (Platform), 'grid' (SunGrid), or 'standalone` (local machine)
    set job_scheduler standalone
    set run_list_maxsize 24
    set normal_queue (bnormal -R rusage[mem=4000])
}
############################
# DEFAULT PINTYPE PARAMETERS
############################

pintype default {
    set total_slew_multiplier 1.0

    set logic_high_name VDD
    set logic_high_threshold 0.7

    set logic_low_name VSS
    set logic_low_threshold 0.3

    set prob_delay_level 0.5

    # Number of slew and load indices 
    # (when importing with -use_default_slews -use_default_loads)

    set numsteps_slew 5
    set numsteps_load 5
    set constraint_numsteps_slew 3

    # Operating load ranges
    set smallest_load 10e-15
    set largest_load 90e-15
    set autorange_load state

    # operating slew ranges
    set smallest_slew 10.0e-12
    set largest_slew 5.0e-9
    set max_tout 5.0e-9

    # Automatically determine largest_load based on max_tout: off or on
    set autorange_load on

    # Noise of points in for noise height
    set numsteps_height 8

    # Input noise width
    set numsteps_width 5

    # driver model: pwl, emulated, active, active-waveform, custom
    set driver_mode emulated 

    # driver cell name (relevant only when driver_mode is `active`)
    # set driver pwl
}

#####################################
# LIBERTY_MODEL_GENERATION_PARAMETERS
#####################################
define_parameters liberty_model {
    # Add Liberty header attributes here for use with "model -create_new_model"
    set delay_model "table_lookup"
    set default_fanout_laod 1.0
    set default_inout_pin_cap 1.0
    set default_input_pin_cap 1.0
    set default_output_pin_cap 0.0
    set default_cell_leakage_power 0.0
    set in_place_swap_mode match_footpoint
}
#######################
# VALIDATION PARAMETERS 
#######################
define_parameter validation {
    # Add validation parameters here
}
```

* __cell.tcl__ - 设置需要re-char的cell列表

```tcl
# replace with names of real cells in your library
set cells {CELL1 CELL2 CELL3}
```

* __run.tcl__ - SiliconSmart启动脚本

```tcl
source ./cell.tcl
set charpoint testcase

set LIB_FILE ${DIGITAL_LIB_DIR}/${LIB}_ccs.lib 
set NETLIST_DIR ./netlist/

create $charpoint
set_log_file $charpoint/sis.log

exec cp configure.tcl ${charpoint}/config/configure.tcl
set_location $charpoint

set_config_opt simulator bisection 1
set_config_opt ccsn_rechar 1

import -fast -liberty ${LIB_FILE} -extention .cdl -netlist_dir ${NETLIST_DIR} $cells
configure -fast -timing -power -ccs -ccs_noise $cells


characterize $cells 

model -verilog -output my_models $cells
model -timing -power -ccs -ccs_noise -output ccs $cells

log_info "IAMDONE"
```

* __Makefile__ 

```makefile
all: clean run

run:
    bsub -Is siliconsmart run.tcl

clean:
    rm -rf ./testcase
```

