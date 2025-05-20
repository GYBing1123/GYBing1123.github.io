+++
date = '2025-05-20T09:31:09+08:00'
draft = false 
title = 'Nvim的基础设定'
tags = ["nvim"]
categories = ["nvim"]
+++
这个文章系列主要记录的是，我搭建NeoVim工作环境的过程。
重点记录以下内容：
- 所用到的工具
- 工具间的协作
- 一些vim使用小技巧
- 工具的配置

# 为什么会选择NeoVim
作为一个重度的键盘爱好者，`vim`可能是我目前所接触到的编程工具中，最能给我带来编程快乐的编辑器了。其次，在今后的学习中，Linux将作为我主要的开发环境，所以掌握`vim`也是重要的技能。


# Vim的按键
## normal模式
**vim命令的基本结构**
`<verb><range>`
- verb,动作。包括复制、删除等
- range,范围。也被叫做`motion`。

### `verb` 动作
- `d` delete,删除。
- `y` yank,复制。
- `p` paste,粘贴。
- `c` change,修改。
- `f` find,在该行中向右查找内容。`F`向左查找
    如：`f:`(向右查找第一个：)
    `2f:`(向右查找第二个：)
    `2F:`(向左查找第一个：)
### `range` 范围
** 1.text object(可操作文本单元）**
- `i` inner, 内部的.
```bash
# 删除括号内的内容，如(hello world)中的'hello world'
di(

# 删除标签内的内容，如<p>hello wrold</p>中的'hello world'
dit
# tip:t在这里是tag（标签）的意思
```
- s sentence,句子.
- p passage,段落.
- t tag,标签.

** 2.f/t 与<verb>的结合 **
- `df:` 向右删除到第一个`:`，包括`:`。
- `dt:` 向右删除到第一个`:`，不包括`:`。

---
## exec模式 
- `:=` 执行一段lua代码并输出结果。
- `:e` edit,打开一个文件。

---
# Nvim的文件多开：Bufferline
在vim中，我们每打开一个文件就会创建一个`buffer`。
- vim中对文件的编辑，都是在`buffer`中进行的
- 在编辑的过程中，原文件并没有被修改
- 只有在保存的时候，原文件的内容才会被修改
- 当一个文件被编辑到一半时，通过`:e <file_name>`打开另一个文件时。该文件则进入`hidden`状态（即放入内存中）

## buffer文件的查看
通过`:buffers`可以查看所有的buffer文件
```bash
:buffers
  1 #h + "nvim.md"                      line 54
  3 %a   "plugins.md"                   line 5
```
- 开头的数字代表着buffer的id，通过`buffer+id`可以切换到指定的buffer
- `a`表示active，`h`代表hidden, `+`代表buffer被修改但没有写入原文件

## buffer的分屏查看
- `vsplit` 左右分屏
- `split` 上下分屏
   - <C-w>h：切换到左侧窗口
   - <C-w>l：切换到右侧窗口
   - <C-w>j：切换到下方窗口
   - <C-w>k：切换到上方窗口
   - <C-w>c：关闭窗口
   - <C-w>o：关闭其他所有窗口

## tab
- `:tabnew` 创建新的 tab，
- `:tabclose` 关闭 tab，
- `:tabs` 列出所有 tab，
- `:tabnext` `:tabprevious` 前后切换 tab。

---
# 寄存器（register)
## 类型
- unnameed register: `""`
- named register: `"a`

查看：`h registers`

## 使用
- 复制当前行到寄存器a：`"ayy`
- 粘贴寄存器a中的内容到指定位置：`"ap`

## 不放入寄存器
直接删除不放入寄存器中
`"_cw`: 直接删除一个单词，并且不影响当前复制、剪切的内容

## 与系统剪贴板互通
可以使用`"+`寄存器器，就可以将neovim中的内容复制到系统剪贴板中。

还可以通过设置neovim来实现。
```lua
vim.opt.clipboard = "unnamedplus" -- unnamed register + plus register
```

---
# 宏操作
- 按q开始录制宏。如`qa`
- 再次按q结束录制宏。
- 按@调用宏。如`@a`

# neovim默认支持数字的自动加减
在 normal mode 下
- 在数字上按下 Ctrl + a 可以让数字加一
- 按下 Ctrl + x 可以减一

---
# 事件:autocmd
## 创建cmd
`vim.api.nvim_create_autocmd(event, opts)`
- event:可以是字符串或者是table，可以通过`:h events`查看。
- opts：当前autocmd的table
    - group: 使用 vim.api.nvim_create_augroup 创建的组，这样我们可以批量处理拥有相同 group 的 autocmd
    - pattern: 事件的模式，取决于具体的事件类型，例如 BufRead 事件可以是 BufRead *.txt，后面的 *.txt 就是模式，即仅对 txt 格式的文件触发 BufRead
    - buffer: 触发事件的 buffer id，如果不设置则为所有 buffer 添加 autocmd；不可以和 pattern 一起使用
    - desc: 对 autocmd 的描述
    - callback: 事件触发的时候执行的函数，该函数接受一个 table 作为传入参数，具体参数想见 :h nvim_create_autocmd 的 event-args 部分
    - once: 是否只触发一次，默认为 false 


## event:FileType事件
Neovim 在打开文件的时候，会检测文件类型，此时就会触发 FileType 事件。

例如，我们想要在打开 python 文件的时候讲缩进设置为 2 个字符，将 colorcolumn 设置在 120 个字符处，就可以这样做：
```lua
vim.api.nvim_create_autocmd("FileType", {
    pattern = "python", -- 对于 FileType 事件，pattern 就是文件类型
    callback = function()
        vim.bo.tabstop = 2
        vim.bo.shiftwidth = 0
        vim.wo.colorcolumn = "120"
    end,
})
```

## 自定义事件
所有的自定义事件其实都属于 User 事件的不同 pattern。

怎么自己创建自定义事件呢？
- 很简单，在特定的时间点发出 event + pattern 就行，这个命令就是 `vim.api.nvim_exec_autocmds`。

譬如说，我想要确保 mason 加载完之后再加载 none-ls，就可以这样做：
```lua
{
    "williamboman/mason.nvim",
    -- 发出event
    event = "VeryLazy",
    config = function (_, opts)
        -- 此处省去之前的配置代码
        vim.api.nvim_exec_autocmds("User", { pattern = "MasonLoaded" })
    end,
}

{
    "nvimtools/none-ls.nvim",
    event = "User MasonLoaded",
    -- 此处略缺剩下的配置代码
}

```

