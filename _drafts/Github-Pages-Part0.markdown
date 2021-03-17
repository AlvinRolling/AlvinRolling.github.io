---
layout: post
title:  "Github Pages 问题总结"
date:   2021-03-17 11:38:30 +0800
categories: Github Pages
---

# 问题总结

1. 新版 Jekyll 不再支持Pygments代码高亮，仅支持Rouge。从目前的情况看，Rouge 对 Tcl 的支持较差。在 `{}` 使用`#` 注释则无法识别，且高亮配色较不友好。
2. 尝试更改代码高亮配色。修改 `_site/assets/main.css` 文件用到的配色方案。但在使用 `bundle exec jekyll serve` 重新开启服务后，发现修改的 `main.css` 会自动复原。这可能是由于设置了 theme，会使得该页面重置到theme默认的设置。目前暂未找到解决方案。
3. 修改theme为 `cayman` 遇到问题，无法启动jekyll服务。Github上有较多类似issue，目前还未尝试解决，仍使用默认 `minima` 主题.
 
