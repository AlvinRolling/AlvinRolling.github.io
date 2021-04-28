---
title:  "Euclide - Part0"
date:   2021-03-15 16:06:30 +0800
categories: EDA Front-End
toc: true
toc_icon: glasses
---

本文介绍Synopsys的前端设计IDE Euclide。

数字电路前端设计一直缺少高效的IDE工具。目前一直使用的文具是Sublime Text + SystemVerilog 插件满足代码高亮，自动例化，层级显示等大部分功能。而VCS编译所需用到的文件使用手动编写。该方法仍有较大不便

1. VCS编译过程中的错误需要手动定位到代码。
2. Lint错误难以及时发现。
3. 编辑器不支持UVM。

尝试过VS Code，在对SV的支持上不如Sublime Text，对UVM的支持上有进步。另外在Eclipse IDE下有[DVT](https://www.dvteclipse.com)付费插件，从介绍上看能够比较好的满足Lint及UVM的需求。

# Euclide
[Euclide](https://www.synopsys.com/verification/ide/euclide.html)的出现终于看到了一丝曙光。从介绍上，应该是最近全新发布的一款IDE，仍然基于Eclipse. 从官网的功能描述上，应该相当好易于上手。
>VCS and Verdi users can easily adopt the solution using existing project files and scripts.

## 安装
不同于其他Synopsys的工具，无需使用Synopsys Installer. Euclide只需要将安装包解压，并将执行文件地址添加到环境变量即可。

## 使用
Euclide使用Compilation Unit Definition（CUD)文件，类似VCS编译时候, -f/-F 参数用到的filelist文件，列出需要编译的文件列表。Euclide的lint、auto-complete功能仅在列出的文件中可用。每个项目至少包含一个 root CUD 文件，CUD文件可以继续包括其他子CUD文件。

但与VCS仍有区别

1. VCS编译命令有`-top`选项，用于指定顶层设计module的名称。CUD文件也有-top选项，但指定的是带路径的文件名，例如 `-top ./src/tb.sv`.
2. CUD 另外包括`-top_synth`等选项。
3. 两者都支持`-F`和`-f`选项。

因此，建议在CUD文件中使用VCS兼容的语法，不使用`-top`选项，并且使用`-F`选项指定另一CUD文件。在VCS的编译选项`-top`可以写在Makefile文件中。

{% highlight Python linenos%}
# Verilog Source File
test.sv

# Other CUD File
# another.cud     ## Not Recommended
-F another.cud    ## Recommended

# Folders
# ./subfolders    ## Not Recommemded
{% endhighlight %}

代码编写上，简单的 `always_comb`, `always_ff` 等模块编写速度仍比Sublime慢，但其提供了更多类型的自动补全模板，可从 `EuclidE Preferences -> Editor -> Templates` 查看。Euclide最吸引人的地方，在于它能够实时显示并定位代码中的问题，下方两图展示了其功能。

![On-the-fly Problem]({{ site.url }}/assets/euclide/prob.jpg)
![On-the-fly Problem2]({{ site.url }}/assets/euclide/prob2.png)


为了实现 UVM 相关功能，需要在 `EuclidE Preferences -> Compiler -> Compilation Unit` 中打开 `Automatically compile UVM library`. 之后，同样使用auto-complete功能。相比Sublime Text和VS Code的UVM功能，Euclide功能更为全面。

![uvm]({{site.url}}/assets/euclide/uvm_autocomplete.gif)

# 后续
更具体的使用方法将在之后陆续学习。从上手的体验来看，在SV代码编写的便捷性上，ST仍然最好，但由于Euclide的代码实时lint与定位功能，对UVM的更好支持，以及与VCS和Verdi的结合，仍然是一款非常优秀的工具。后续使用上，应该在代码编写阶段使用ST，之后使用Euclide进行代码的Lint与Debug。

