+++
date = '2025-05-21T17:54:37+08:00'
draft = false
title = 'Neovim 的 Lsp 配置'
categories = ["nvim"]
tags = ["nvim", "lsp"]
+++
# LSP 是什么
Language Server Protocol，语言服务器协议，在开发工具与语言服务器之间交换消息，实现语言相关的功能（如代码补全）和编辑器本身解耦。可以实现在一个编辑器（VScode或Neovim）上写多种语言的代码。

## LSP 功能
- 编辑器打开一个文件的时候，就会启动一个 language server，此时文档内容就会被保存在这个语言服务器上。
- 文档内容进行修改的时候，向语言服务器发送相应信号，服务器端会对存储的文档内容进行更新并重新进行分析,将 error 和 warning 信息返回给编辑器；
- 执行跳转功能的时候，向语言服务器发送请求，语言服务器会将跳转的目标位置（行、列等）返回给编辑器；
- 请求补全的时候，语言服务器也会结合服务器上存储的文档内容进行上下文分析，返回补全建议。

## LSP 插件
LSP 只是一个协议，并不主动提供以上功能。Neovim 还需要其他插件完成与语言服务器之间的信息交换。

具体需要的插件有：
- `mason.nvim` —— 语言服务器的管理器。。
- `nvim-lspconfig` —— 为语言服务器提供默认的客户端配置。
- `mason-lspconfig.nvim` —— 解决 mason 与 nvim-lspconfig 语言服务器名称不统一的问题
- `bufferline.nvim` —— 通过配置将错误信息的统计结果，显示到buffer栏上
- `blink.cmp` —— 代码补全插件

