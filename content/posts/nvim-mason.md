+++
date = '2025-05-22T10:27:33+08:00'
draft = true
title = 'Nvim Mason'
categroies = ["nvim"]
tags = ["nvim", "mason"]
+++

# Mason 是什么
Mason 是语言服务器的管理工具。它对于语言服务器的管理，类似 Lazy 对 nvim 插件的管理。

# Mason 的安装
在 nvim/lua/lsp/ 文件夹中新建 `mason.lua` 文件，并将 GitHub 中的代码复制到文件中。

```lua
return{
    "mason-org/mason.nvim",
    opts = {}
}
```

安装完成后，可以通过在 command-line 输入 `:Mason<CR>` 来查看。

# 代码编辑（以 lua 为例）

## 1. 安装所需语言服务器
```lua
return{
    "mason-org/mason.nvim",
    -- 设置懒加载
    event = "VeryLazy",
    opts = {},

    -- registry 是 mason 的服务列表。第一次运行 mason 时会从服务器获取这个列表
    local registry = require "mason-registry"

    -- 安装所需的语言服务器
    local success, package = pcall(registry.get_package, "lua-language-server")
    -- 判断语句，所需语言在服务列表中且没有安装过，才会执行安装
    if success and not package:is_installed() then
        package:install()
    end

}
```

**以上代码中使用到了 lua 的 pcall 函数**
pcall(function, arg1, ...) 是一个实现错误处理的函数。它会执行 function(arg1, ...)函数，并返回两个值。
- 第一个值为boole类型，表示函数是否执行成功。
- 第二个值为函数的返回值或错误信息。

