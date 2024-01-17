---
title:  "Vim UVM Snippets"
date:   2024-01-17 12:00:30 +0800
categories: Vim
toc: true
toc_icon: glasses
# layout: posts
---

This article outlines the process of adding UVM snippets to the `vim-snippet` GitHub repository.

## Introduction

### Background

Vim snippets auto-completion is commonly facilitated by two plugins: [UltiSnips](https://github.com/SirVer/ultisnips/) and [vim-snippets](https://github.com/honza/vim-snippets). Currently, there is no support for UVM snippets inside the *vim-snippets* repository. However, another VS Code plugin, [SystemVerilog-1800-2012](https://github.com/gvekony/sv-1800-2012), provides useful UVM snippets, although it hasn't been updated for more than 2 years.

### Target

This article guides you through the process of migrating UVM snippets from the VS Code plugin to the **vim-snippets** repository, enabling their use within Vim through the **UltiSnips** plugin.

## Process

While both UltiSnips and snipMate formats are supported, **vim-snippets** suggests using **snipMate** for its wider adoption. Additionally, there are bug-related issues with the '`' escape character in the UltiSnips format. Given the wide use of this character in UVM macros, we ultimately opted for the **snipMate** format.

### VS Code Plugin Snippets

Fetch the snippets code from [here](https://github.com/gvekony/sv-1800-2012/blob/master/snippets/uvm_containers.json), and then replace `\n\n$0` with `$0\n\n` to avoid the format error. *snipMate* requires extra tabs at the beginning of `$0`. Otherwise, the following conversion step would not be able to do so.

### Snippet Format Conversion

An example of converting `uvm_container.json` into `uvm_container.snippet` is shown below.

```python
import json

json_file = './uvm_container.json'
output_file = './uvm_container.snippet'

with open(json_file,'r') as load_f:
    decoded_hand = json.load(load_f)
    load_f.close()

with open(output_file,'w') as output_f:
    for key in decoded_hand.keys():
        body   = decoded_hand[key]["body"]
        prefix = decoded_hand[key]["prefix"]

        output_f.write('snippet ')
        output_f.write(prefix+'\n\t')

        body = "\n\t".join(body)
        output_f.write(body)
        output_f.write('\n\n')

    output_f.close()
```

Below is the first class snippet.

```verilog
snippet uvm_object
  //  Class: $1
  //
  class ${1:$TM_FILENAME_BASE} extends ${2:uvm_object};
    `uvm_object_utils($1);

    //  Group: Variables


    //  Group: Constraints


    //  Group: Functions

    //  Constructor: new
    function new(string name = "$1");
      super.new(name);
    endfunction: new
    $0
  endclass: $1
```

Before adding it to the file under the vim plugin `vim-snippet/snippets/systemverilog.snippet`, two additional steps need to be taken care of:

- Replace `$TM_FILENAME_BASE` with a default class name, such as `my_class`.
- Replace \`uvm_* with \\\`uvm_*, as the \` symbol is used to trigger the [shell code](https://github.com/SirVer/ultisnips/blob/b393ba65386d47664421e1f8b246a87a6e8b218c/doc/UltiSnips.txt#L851), and it should be used with an [escape character](https://github.com/SirVer/ultisnips/blob/b393ba65386d47664421e1f8b246a87a6e8b218c/doc/UltiSnips.txt#L753).

Now that we have the completed snippets for `uvm_container`, append them to the end of `systemverilog.snippet`. Afterward, open a .sv file in Vim to test their functionality.

![uvm_snippet_test]({{site_url}}/assets/Vim/uvm_snippet_test.png)

### Github Pull Request

A [pull request](https://github.com/honza/vim-snippets/pull/1510) is opened to merge the changes into the *vim-snippet* repository.

## TODO

- Refine the UVM snippets, addressing any inclusion of unwanted methods in certain class snippets.
- Add missing class snippets, such as `uvm_driver`, `uvm_monitor`, etc. Presently, only basic `uvm_object` and `uvm_component` related class snippets are supported.
- Repeat similar process for `uvm_macros` and `uvm_phase` snippets provided by the original [VS Code Plugin](https://github.com/gvekony/sv-1800-2012/tree/master/snippets).
