+++
date = '2025-05-23T08:56:19+08:00'
draft = true
title = 'neovim 代码补全'
categories = ["nvim"]
tags = ["nvim", "blink.cmp"]
+++

# Neovim 如何实现代码的补全
Neovim 的代码补全功能，实际上还是通过 LSP 实现的。
LSP 服务器端根据光标位置计算并返回补全结果，但结果呈现出来、在浏览这些补全项的时候显示相应的文档以及其他一些更复杂的功能，还是需要依赖补全引擎。
目前最主流的补全引擎是 `nvim/blink.cmp`

# blink.cmp 补全引擎的安装

