+++
date = '2025-05-21T12:37:38+08:00'
draft = false
title = 'neovim 插件'
categories = ["nvim"]
tags = ["nvim", "bufferline", "autopairs", "surround"]
+++
该文章主要记录一些常用的插件，基本上都存放在 nvim/lua/plugins/ 文件夹中，以下都会简写为 plugins/。

---
# 管理 Buffer —— bufferline.nvim
## 安装
在 plugins/ 文件夹中新建 `bufferline.lua` 文件。并从 GitHub 上复制安装代码。
```lua
return{
    'akinsho/bufferline.nvim',
    version = "*",
    dependencies = {
        'nvim-tree/nvim-web-devicons'
    },
    -- 为了让bufferline插件生效，加入 opts 参数
    opts = {}
}
```

## 优化
1. bufferline 插件通过 <command-line mode> 中的命令行进行 buffer 间的切换。复杂且不易操作，可以在配置文件中设置快捷键，提升使用体验。
2. 因为引入了属性 keys，会触发 Lazy 的懒加载模式，使得 bufferline 在使用快捷键时才进行加载。 所以需要加入 lazy = false， 关闭 Lazy 的懒加载模式

优化后的代码如下：
```lua
return{
    'akinsho/bufferline.nvim', 
    version = "*", 
    dependencies = {
        'nvim-tree/nvim-web-devicons'
    },
    opts = {},
    -- 关闭懒加载模式
    lazy = false,
    -- 设置快捷键，silent = true 代表执行完命令后清空<command-line>
    keys = {
        { "<leader>bh", ":BufferLineCyclePrev<CR>", silent = true },
        { "<leader>bl", ":BufferLineCycleNext<CR>", silent = true },
        { "<leader>bd", ":bdelete<CR>", silent = true },
        { "<leader>bo", ":BufferLineCloseOthers<CR>", silent = true },
        { "<leader>bp", ":BufferLinePick<CR>", silent = true },
        { "<leader>bc", ":BufferLinePickClose<CR>", silent = true },
    },

}
```

## 基于 LSP 功能的扩展
在成功安装了 LSP 服务的基础上，可以对 bufferline 插件中的 opst 字段下的 `options` 进行设置。可以实现将错误信息的统计结果显示在 buffer 栏的效果。

具体代码如下：
```lua
return{
    'akinsho/bufferline.nvim',
    lazy = false,
    version = "*",
    dependencies = {
        'nvim-tree/nvim-web-devicons'
    },
    -- 添加设置
    opts = {
        options = {
            -- nvim_lsp 是 Neovim 内置的 LSP 诊断系统
            diagnostics = "nvim_lsp",
            diagnostics_indicator = function (_, _, diagnostics_dict, _)
                local indicator = " "
                for level, number in pairs(diagnostics_dict) do
                    local symbol
                    if level == "error" then
                        symbol = " "
                    elseif level == "warning" then
                        symbol = " "
                    else
                        symbol = " "
                    end
                    indicator = indicator .. number .. symbol
                end
                return indicator
            end
        }
    },
    keys = {
        { "<leader>bh", ":BufferLineCyclePrev<CR>", silent = true },
        { "<leader>bl", ":BufferLineCycleNext<CR>", silent = true },
        { "<leader>bd", ":bdelete<CR>", silent = true },
        { "<leader>bo", ":BufferLineCloseOthers<CR>", silent = true },
        { "<leader>bp", ":BufferLinePick<CR>", silent = true },
        { "<leader>bc", ":BufferLinePickClose<CR>", silent = true },
    },
}
```

---
# 自动补全 —— nvim-autopairs
对各类符号进行自动补全
```lua
{
    'windwp/nvim-autopairs',
    event = "InsertEnter",
    config = true,
    -- use opts = {} for passing setup options
    opts = {},
    -- this is equivalent to setup({}) function
}
```

---
# 添加标记 —— nvim-surround
在指定字符串两边加上 `括号/大括号/<tag>` 等标记。

## 使用方法
在原来的 <verb> 后s，具体使用如下：

|原命令|含义| surround 命令|含义|
|:--:|:--|:--:|:--|
|`yiw`|复制单词|`ysiw"`|在单词的两边加上双引号|
| | |`ysiwt`|在单词的两边加上tag标记|
|`d`|删除|`ds"`|删除最近的一对双引号|
|`c`|修改|`cs"t`|修改最近一对双引号为tag标记,也可以修改为其他符号|
|<visual mode>|视图模式|`S"`|为视图模式下选择内容的两边，加上双引号|


## 安装
```lua
return{
    "kylechui/nvim-surround",
    version = "^3.0.0", -- Use for stability; omit to use `main` branch for the latest features
    event = "VeryLazy",
    config = function()
        require("nvim-surround").setup({
            -- Configuration here, or leave empty to use defaults
        })
    end
}

```
---
# nvim-tree


---
# nvim-lualine


---
# indent-blankline


---
# telescope

