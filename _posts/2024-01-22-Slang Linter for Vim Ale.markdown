---
title:  "Integrating Slang Linter into the Vim ALE"
date:   2024-01-22 12:00:30 +0800
categories: Vim
toc: true
toc_icon: glasses
# layout: posts
---

This article outlines the process of incorporating [slang](https://github.com/MikePopoloski/slang) linting tools into the Vim plugin [ale](https://github.com/dense-analysis/ale).

## Introduction

As described in a prior [article](https://alvinrolling.github.io/eda/uvm/Vim_SV_UVM/), there are numerous tools available for RTL parsing and linting. Among them, **slang** stands out for its exceptional performance on the [sv-tests](https://github.com/chipsalliance/sv-tests), exhibiting an almost 100% pass rate. Furthermore, **slang** offers the following advantages compared to other counterparts.

- The [command line options](https://sv-lang.com/command-line-ref.html) exhibit some similarities to common EDA tools (especially VCS).
- The output lint messages bear resemblance to **clang** results.
- Actively maintained and developed with rich [documentation](https://sv-lang.com/).
- Support for **UVM**.

The following 2 figures show the slang output message through command line and the results of ALE integration in Vim.

![slang_lint_message]({{site_url}}/assets/Vim/slang_lint_message.png)
*Lint Output Message from Slang*

![slang_vim_ale]({{site_url}}/assets/Vim/slang_vim_ale.png)
*Slang integration with Vim ALE*

### ALE Plugin

Vim ALE (Asynchronous Lint Engine) plugin is a powerful tool for real-time linting and fixing code in Vim.
It supports various programming languages, providing on-the-fly feedback and suggestions to improve code quality. ALE runs asynchronously, ensuring minimal disruption to your workflow while offering comprehensive linting capabilities.

At present, ALE supports Verilog linters such as hdl_checker, iverilog, verilator, vlog, xvlog, and yosys. In the subsequent section, I will incorporate support for the **slang** tool into the ALE plugin.

## Introducing slang to ale

Finally, check your development complies with the guidance by running the `./run-tests` scripts.

To introduce the new Slang linter to ALE, the following three steps are required:

1. Write a `slang.vim` file to specify a *Getcommand* and a *handler* function to convert the output messages to the ALE structure.
2. Write two test files to ensure that the *Getcommand* and *handler* functions work as expected.
3. Update the doc files accordingly.

Finally, confirm that your development complies with the guidelines by running the `./run-tests` script.


### slang.vim

The `slang.vim` is located under `${ale_root}/ale_linters/verilog/`. The source code is as follows.
<script src="https://gist.github.com/AlvinRolling/2eb9b968eeb73a3504a5d351009a7b68.js"></script>

The `GetCommand` function requires explanation. For `%s:h`, `%s` represents [the path to the current file](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale.txt#L4524-L4526), and `:h` is a [filename modifier](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale.txt#L4555-L4563). `%t` refers to the path of a [temporary file](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale.txt#L4531-L4542).

The `pattern` expression is adapted from `clang.vim`, which further references [gcc.vim](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/autoload/ale/handlers/gcc.vim#L104), as the **slang** output is similar to **clang**. The `RemoveUnicodeQuotes` function requires further refinement.

Currently the `filename` keyword is not used. May fail to work when this is added to the `l.item`.
{: .notice--warning}

### test files

Follow this [development guide](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale-development.txt#L475). Two test files have been added under the test directory as outlined below.

<script src="https://gist.github.com/AlvinRolling/df2e06e625405482359da04755a0021e.js"></script>

### Language specific doc

Update the documentations for the **slang** linter.

<script src="https://gist.github.com/AlvinRolling/1327ec19e37e1f881884da507362af3f.js"></script>

The `yosys` typo and a missing period `.` symbol is added as well.
{: .notice--note}

### run tests

Now, execute `./run-tests` and redirect the output to a text file to examine the results. Ensure that:

- All tests have passed, verified by the absence of `(x)` symbols in the outputs.
- All 5 `--linter-only` checks have passed.

Once all tests are successful, specify the linter for the SystemVerilog filetype using `let g:ale_linters = {'systemverilog' : ['slang']}` in your `.vimrc`. The integration process is now complete, and a pull request has been submitted.

## Summary

While many ASIC engineers are accustomed to utilizing commercial tools such as **VCS** for linting and parsing, a Vim-based custom solution may present advantages for swift responses. It's noteworthy that some vendors are striving to offer more efficient editor solutions, such as [EuclidE]({{site_url}}/eda/front-end/EuclidE-Part0/), tailored for front-end designers. Nevertheless, these tools are far from being user-friendly.

This article has expanded my understanding of utilizing and customizing Vim features. Despite the **slang** linter not being flawless at this juncture, it has already enhanced the editing workflow, and I am optimistic about its continuous development.

### TODO

- Currently, the `note` messages are treated as warnings; they will be separated later.
- The linting is currently tested on a single file; project configuration needs to undergo testing.
