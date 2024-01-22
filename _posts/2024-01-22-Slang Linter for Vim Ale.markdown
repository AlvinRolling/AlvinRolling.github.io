---
title:  "Integrating Slang Linter into the Vim ALE"
date:   2024-01-22 12:00:30 +0800
categories: Vim
toc: true
toc_icon: glasses
# layout: posts
---

This article outlines the process of incorporating [slang](https://github.com/MikePopoloski/slang) tools into the Vim plugin [ale](https://github.com/dense-analysis/ale).

## Introduction

As described in a prior [article], there are numerous tools available for RTL parsing and linting. Among them, **slang** stands out for its exceptional performance on the [sv-tests](https://github.com/chipsalliance/sv-tests), exhibiting an almost 100% pass rate. Furthermore, **slang** offers the following advantages compared to other counterparts.

- The [command line options](https://sv-lang.com/command-line-ref.html) exhibit some similarities to common EDA tools (especially VCS).
- The output lint messages bear resemblance to **clang** results.
- Actively maintained and developed with rich [documentation](https://sv-lang.com/).
- Support for **UVM**.

![slang_lint_message]({{site_url}}/assets/Vim/slang_lint_message.png)
*Lint Output Message from Slang*

### ALE Plugin

Vim ALE (Asynchronous Lint Engine) plugin is a powerful tool for real-time linting and fixing code in Vim.
It supports various programming languages, providing on-the-fly feedback and suggestions to improve code quality. ALE runs asynchronously, ensuring minimal disruption to your workflow while offering comprehensive linting capabilities.

At present, ALE supports Verilog linters such as hdl_checker, iverilog, verilator, vlog, xvlog, and yosys. In the subsequent section, I will incorporate support for the **slang** tool into the ALE plugin.

## Introducing slang to ale

To integrate **slang** with ALE, three types of files need to be updated accordingly. You can refer to [this pull request](https://github.com/dense-analysis/ale/pull/3500/files) for **verible** linter as a guide

### slang.vim

The `slang.vim` is located under `${ale_root}/ale_linters/verilog/`. The source code is as follows.
<script src="https://gist.github.com/AlvinRolling/2eb9b968eeb73a3504a5d351009a7b68.js"></script>

The `GetCommand` function requires explanation. For `%s:h`, `%s` represents [the path to the current file](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale.txt#L4524-L4526), and `:h` is a [filename modifier](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale.txt#L4555-L4563). `%t` refers to the path of a [temporary file](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale.txt#L4531-L4542).

The `pattern` expression is adapted from `clang.vim`, which further references [gcc.vim](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/autoload/ale/handlers/gcc.vim#L104), as the **slang** output is similar to **clang**. The `RemoveUnicodeQuotes` function requires further refinement.

### test files

TODO: Follow this [development guide](https://github.com/dense-analysis/ale/blob/8922478a83cd06bfe5b82eb45279649adc4ec046/doc/ale-development.txt#L475).

### Language specific doc

Update the documentation in `doc/ale-verilog.txt` for the **slang** linter.

### run tests

TODO: The built-in run-tests utilize Docker. However, there seems to be a network VPN issue with downloading the Vim repository inside Docker.

## Summary

At present, the integration of the Slang linter into ALE has been tested on local machines. However, due to networking issues with Docker, which I am not thoroughly familiar with, the test files haven't been executed and developed. Therefore, further progress of raising a pull request and subsequent merging into the ALE repository is still pending.
