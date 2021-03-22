---
layout: post
title:  "RISC-V IP & IDE 总结"
date:   2021-03-22 15:20:30 +0800
categories: RISC-V
---

由于开源 RISC-V Core 缺少IDE，软件开发有诸多不便。后续准备将现有使用的基于 [IBEX](https://github.com/lowRISC/ibex) 的 RISC-V 核心替换为商用IP。国内外常用的RISC-V IP&IDE Vendor 列出如下。由于国内 Starfive 曾为 Sifive 的国内分部，下表中未单独列出。

| Vendor           | Open Source Core | Core Name | Commercial Core | Core Name                                 | IDE | IDE Name       | IDE Mac Compatible | Custom Instr. | Language | Comp. Location       | Comments                                       | Ref Link                                                |
|------------------|------------------|-----------|-----------------|-------------------------------------------|-----|----------------|--------------------|---------------|----------|----------------------|------------------------------------------------|---------------------------------------------------------|
| **Andes Technology** | NO               | -         | YES             | V5 Cores (A Series, N/D Series, V-Series) | YES | AndeSight      | NO                 | YES           | -        | Hsinchu City, Taiwan |                                                | [AndesTech](http://www.andestech.com/en/risc-v-andes/)               |
| **Ashling**         | NO               | -         | NO              | -                                         | YES | RiscFree       | -                  | NO            | -        | VA, USA              | Multiple 3-rd Party Core Supported             | [Ashling](https://www.ashling.com/ashling-riscv/)                  |
| **Codasip**          | NO               | -         | YES             | 1/3/5/7 Series                            | YES | Codasip Studio | -                  | YES           | -        | Brno, Czech          |                                                | [Codasip](https://codasip.com/products/codasip-risc-v-processors/) |
| **Efinix**           | NO               | -         | YES             | Opal/Jade/Ruby SoC                        | YES | -              | -                  | NO            | -        | CA, USA              | FPGA Company                                   | [Efinix](https://www.efinixinc.com/products-riscv.html)           |
| **Imperas**          | NO               | -         | NO              | -                                         | NO  | -              | -                  | YES           | -        | Oxfordshire U.K.     | Individual Solutions / 3-rd Party IP Supported | [Imperas](https://www.imperas.com/imperas-riscv-solutions)         |
| **Nuclei (芯来科技)**    | NO               | -         | YES             | N/NX/UX Class                             | YES | Nuclei Studio  | NO                 | YES           | -        | Shanghai China       |                                                | [Nuclei](https://www.nucleisys.com/product.php)                   |
| **SiFive**           | NO               | -         | YES             | E/S/U Cores                               | YES | Freedom Studio | YES                | YES           | -        | CA, USA              | From UC Berkeley                               | [SiFive](https://www.sifive.com/risc-v-core-ip)                   |
| **Syntacore**        | YES              | SCR1      | YES             | SCR3/4/5/7                                | YES | -              | NO                 | YES           | -        | Moscow, Russia       |                                                | [Syntacore](https://syntacore.com/page/products/processor-ip)        |
