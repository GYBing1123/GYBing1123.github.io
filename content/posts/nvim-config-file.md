+++
date = '2025-05-20T11:32:49+08:00'
draft = false
title = 'Nvim基础配置文件'
categories = ["nvim"]
tags = ["nvim", "配置文件", "basic", "keymap"]
+++

# Neovim的配置文件的位置
在neovim中输入以下命令,确定配置文件的位置
```base
# 查看配置文件的位置
:=vim.fn.stdpath("config")

# 查看数据文件的位置
:=vim.fn.stdpath("data")
```
## `:=`与`:lua`的区别
- `:=`：执行一段 lua 代码，并输出结果。
- `:lua`：执行一段lua代码，但不输出结果。

---
# Neovim配置文件的格式
- nvim
    - init.lua
    - lua/
        - core/
        - plugins/
        - lsp/

---
# 基础配置文件：`lua/core/basic.lua`
```lua
-- 显示行号
vim.opt.number = true
-- 显示相对行号
vim.opt.relativenumber = true

-- 光标所在行高亮
vim.opt.cursorline = true

-- 将tab键转换为空格
vim.opt.expandtab = true
-- tab键所占的空格数
vim.opt.tabstop = 4
-- 搭配上一个参数，neovim 默认启用了 smarttab 选项，该选项的作用之一就是，在开头敲下 Tab 时，添加 shiftwidth 个空格
vim.opt.shiftwidth = 0

-- 文件被其他外部程序修改了之后，neovim 会自动重新加载
vim.opt.autoread = true

-- 显示特殊字符
vim.opt.list = true
-- 将tab与空格显示为其他字符
vim.opt.listchars = {tab = ">-", trail = "-"}
```

---
# Neovim配置快捷键的格式
```lua
vim.keymap.set(mode,lhs,rhs,opts)
```
## mode：模式
`mode`-快捷键的生效模式。可以是单字符串，也可以是table。
  - `n`(normal mode)
  - `i`(insert mode)
  - `c`(command-line mode)
## lhs：键位
`lhs`-快捷键的按键
    - `<C-a>`:表示Ctrl+a
    - `<A-b>`:表示Alt+b
### <leader>
自定义快捷键的前缀`leader key`
```lua
-- 自定义快捷键前缀为空格,绑定快捷键的时候可以用 <leader> 表示。
vim.g.mapleader = " "
vim.keymap.set("n", "<leader>aa", ":lua print(123)<CR>", {})
```
## rhs：实现功能
`rhs`:绑定的功能。可以是另一组快捷键，也可以是一个lua函数。

### rhs的值为lua函数
```lua
vim.keymap.set({ "n", "i" }, "<C-a>b", 
    function ()
        print("hello world")
    end, 
    { silent = true })
```
- lua语法的函数以`function`关键字声明，结尾加一个`end`.
## opts：额外设置
`opts`:table,包含对这个快捷键的额外设置。

快捷键的属性。
- `remap`:值为布尔值。默认是`false`(禁用递归映射)
- `silent`:值为布尔值。`true`(command-line mode下输出的命令在执行后，会被清空)
- `nowait`:值为布尔值。`true`(按下快捷键后不等待，立刻执行)
- `desc`:注释

>例子：
>```lua
>vim.keymap.set("n", "<C-a>b", ":lua print('hello world')<CR>", { silent = true })
>-- n，普通视图模式下生效。
>-- <C-a>,按下Ctrl+a后，再按下b键。
>-- lua..<CR>,调用lua函数输出hello world。
>-- silent.., 
>```


## 关于table
table，类似于array和object的混合，即其中的元素可以没有键名也可以是键值对，此时快捷键对多个模式生效

**mode参数的table**
>例:若想在inset mode下也能生效。可以扩展mode的参数为table类型。
>```lua
>vim.keymap.set({"n","i"},"<C-a>b","<Cmd>lua print('hello world')<CR>",{slient = true})
>```
- `<Cmd>`:即无视不同模式直接进入command-line mode

