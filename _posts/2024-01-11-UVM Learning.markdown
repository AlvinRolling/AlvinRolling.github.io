---
title:  "UVM Learning Path"
date:   2024-01-11 11:06:30 +0800
categories: EDA UVM
toc: true
toc_icon: glasses
# layout: posts
---

This article provides a review of UVM learning resources, complete example projects, and useful tools to expedite the learning process.

## Learning Resources Summary

Thanks to the generosity of the Accellera organization, the UVM source code is now publicly accessible. This accessibility has nurtured a surplus of online learning resources, many of which are freely available. Notable websites include [ChipVerify](https://www.chipverify.com/), [Verification Academy](https://verificationacademy.com/), [Duolos](https://www.doulos.com/), [Testbench4u](https://testbench4u.com/), [TestbenchIn](http://www.testbench.in/), and [VerificationGuide](https://verificationguide.com/).

Additionally, a Chinese textbook offers examples that can be accessed [here](https://github.com/shtzw965/puvm), although it may be considered overly detailed. For those seeking an alternative, the Verification Academy provides comprehensive examples, albeit requiring manual downloading from its website.

> To begin your UVM journey, I recommend starting with a foundational introduction from **ChipVerify** and then delving into the UVM cookbook provided by the **Verification Academy** for more in-depth insights. In contrast, **Testbench4u** offers a wealth of FAQs, which can be valuable as you progress in your learning journey. **Verification Academy** and **Duolos** might delve into intricate details and are best suited as reference materials.

## RISC-V UVM repo

Although UVM may appear manageable at the component level, tackling complete examples reveals its escalating difficulty. The intricacy becomes even more pronounced when delving into larger projects. Not a lot of these projects are available online, but we could start with the RISC-V related projects thanks to its open-source nature.

Two noteworthy projects are available:

- [core-v-verif](https://github.com/openhwgroup/core-v-verif)
- [riscv-dv](https://github.com/chipsalliance/riscv-dv?tab=readme-ov-file)

The RISCV-DV project serves as an instruction generator designed for RISC-V processor verification. In the case of Core-V-Verif, it utilizes the RISCV-DV project as a library for verification across multiple cores. It leverages various other libraries simultaneously. The relationship between these two projects is discussed [here](https://docs.openhwgroup.org/projects/core-v-verif/en/latest/corev_dv.html).

It's worth mentioning that both projects, [core-v-verif](https://github.com/openhwgroup/core-v-verif) and [riscv-dv](https://github.com/chipsalliance/riscv-dv?tab=readme-ov-file), may be a bit incomplete in terms of documentation. Engaging with these projects might require extensive self-learning, and you may encounter bugs along the way.

## UVM Editor

Given that UVM is not as widely recognized as other software programming languages, finding a suitable editor may not be as convenient. Options include *Sublime Text* and *Visual Studio Code*. [DVT Eclipse](https://www.dvteclipse.com/) offers additional features but requires commercial licenses.

> Personally, I would still recommend using Vim, and I'm currently working on its integration. Check the [other article]({{site.url}}/eda/uvm/Vim_SV_UVM/) for the integration process.

## UVM Generator

As a beginner, you might be intrigued to discover that EDA tools can automatically generate UVM framework code. Siemens' **Verification Academy** offers a variety of courses on its features, and I believe Synopsys has a similar tool. In a [previous article]({{site.url}}/eda/uvm/ralgen/), I discussed the RAL generator, and a future article may delve into the UVM generator.