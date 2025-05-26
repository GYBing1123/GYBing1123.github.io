+++
date = '2025-05-22T10:27:33+08:00'
draft = false
title = 'Neovim Mason'
categroies = ["nvim"]
tags = ["nvim", "mason", "nvim-lspconfig", "mason-lspconfig"]
+++

# Mason 是什么
Mason 是语言服务器的管理工具。它对于语言服务器的管理，类似 Lazy 对 nvim 插件的管理。

Mason 只提供语言服务器的下载， LSP 功能的实现还需要其他插件的配合：
- nvim-lspconfig —— 负责配置和启动 LSP 服务器。
- mason-lspconfig —— mason 与 lspconfig 之间的桥梁。


---
# Mason 的安装
在 nvim/lua/lsp/ 文件夹中新建 `mason.lua` 文件，并将 GitHub 中的代码复制到文件中。

```lua
return{
    "mason-org/mason.nvim",
    opts = {}
}
```

安装完成后，可以通过在 command-line 输入 `:Mason<CR>` 来查看。

---
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
>pcall(function, arg1, ...) 是一个实现错误处理的函数。它会执行 function(arg1, ...)函数，并返回两个值。
>- 第一个值为boole类型，表示函数是否执行成功。
>- 第二个值为函数的返回值或错误信息。

## 2. 通过 lspconfig 配置和使用 LSP
通过 nvim-lspconfig 插件，配置和使用 LSP 功能。这个插件提供了各种语言的预设模板，让用户可以启动不用语言的 LSP 功能。

nvim-lspconfig 插件的功能是和 mason 插件绑定使用的，所以 nvim-lspconfig 插件的安装，可以通过 `dependencies` 实现。

```lua
return {
    "mason-org/mason.nvim",
    event = "VeryLazy",
    -- 通过dependencies安装 nvim-lspconfig
    dependencies = {
        "neovim/nvim-lspconfig",
    },
}
```
该插件的通过 `require("lspconfig")[<lsp-name].setup(<config>)` 语句调用。

但 `nvim-lspconfig` 插件和 `mason` 插件对同一个 LSP 服务器的命名方式是不一样的。
>比如：mason 中的 `lua-language-server`，在 nvim-lspconfig 中的命名为 `lua_ls`。

这时就要引入另一个插件 mason-lspconfig

## 3. 通过 mason-lspconfig 建立 mason 与 lspconfig 之间的联系
mason-lspconfig 的安装，也可通过 `dependencies` 方法实现。

mason-lspconfig 通过 `require("mason-lspconfig).get_mappings().package_to_lspconfig["lsp-name]` 语句完成 mason 语言名到 lspconfig 语言名的转换。

具体实现代码如下：
```lua
return {
    "mason-org/mason.nvim",
    event = "VeryLazy",
    dependencies = {
        "neovim/nvim-lspconfig",
        "mason-org/mason-lspconfig"
    },
    opts = {},
    config = function (_, opts)
        require("mason").setup(opts)
        local registry = require "mason-registry"

        local success, package = pcall(registry.get_package, "lua-language-server")
        if success and not package:is_installed() then
            package:install()
        end

        local nvim_lsp = require("mason-lspconfig").get_mappings().package_to_lspconfig["lua-language-server"]
        require("lspconfig")[nvim_lsp].setup({})
    end,
}
```

## 4. 针对懒加载的调整。

**问题描述**
当我们按照上面的配置进行保存后，通过 lspconfig 的 `LspInfo` 命令进行 LSP 状态查看时，会发现 LSP 服务器并没有运行。

**原因分析**
出现以上问题的原因是由插件的懒加载造成的,具体来说：
1. Neovim 打开文件时，其实是打开了一个 buffer。 Neovim 会在打开 buffer 时检测文件类型，该检测会触发 `FileType` 事件。
2. `VeryLazy` 类型的插件是在所有事件都加载后，才最后加载的。
3. 根据以上代码， nvim-lspconfig 更是在 mason 被加载到 buffer 上后才被引用的。

综上所述，我们的启动顺序为 `打开文件，触发 FileType 事件` —— `启动 mason 插件` —— `引用 nvim-lspconfig`， 这时 `lspconfig` 是无法处理当前的 buffer 的。
验证方法为：通过 `:e <file-name>` 命令打开一个新的文件， 再输入 `LspInfo` 命令，会发现 LSP 服务器正常运行。

**解决办法**
以上问题，可以通过 `lspconfig` 提供的另一个命令 `LspStart` 来解决。该命令会为当前的 buffer 启动所需的 LSP 服务。

具体代码如下：
```lua
return {
    "mason-org/mason.nvim",
    event = "VeryLazy",
    dependencies = {
        "neovim/nvim-lspconfig",
        "mason-org/mason-lspconfig"
    },
    opts = {},
    config = function (_, opts)
        require("mason").setup(opts)
        local registry = require "mason-registry"

        local success, package = pcall(registry.get_package, "lua-language-server")
        if success and not package:is_installed() then
            package:install()
        end

        local nvim_lsp = require("mason-lspconfig").get_mappings().package_to_lspconfig["lua-language-server"]
        require("lspconfig")[nvim_lsp].setup({})

        -- 手动为当前buffer启动LSP服务
        vim.cmd("LspStart")
    end,
}
```

## 5. Neovim 自带的 LSP 功能
Neovim 本身自带有一部分的 LSP 功能，但这些功能还是要依赖 LSP 服务器的。 当 LSP 服务器被正确加载后， 该部分功能就会显现出来。

我们可以利用其中的一部分优化我们的配置。
```lua
return {
    "mason-org/mason.nvim",
    event = "VeryLazy",
    dependencies = {
        "neovim/nvim-lspconfig",
        "mason-org/mason-lspconfig"
    },
    opts = {},
    config = function (_, opts)
        require("mason").setup(opts)
        local registry = require "mason-registry"

        local success, package = pcall(registry.get_package, "lua-language-server")
        if success and not package:is_installed() then
            package:install()
        end

        local nvim_lsp = require("mason-lspconfig").get_mappings().package_to_lspconfig["lua-language-server"]
        require("lspconfig")[nvim_lsp].setup({})

        vim.cmd("LspStart")
        -- 利用 neovim 自带的 api 配置诊断的显示结果。
        vim.diagnostic.config({
            virtual_text = true,
        })
    end,
}
```
## 6. 代码优化

```lua
return{
    "mason-org/mason.nvim",
    event = "VeryLazy",
    opts = {},
    dependencies = {
        "neovim/nvim-lspconfig",
        "mason-org/mason-lspconfig.nvim"
    },


    config = function(_, opts)
        require("mason").setup(opts)
        local registry = require("mason-registry")

        -- 自定义setup函数,通过函数安装和配置 LSP 服务器
        local function setup(name, config)
            local success, package = pcall(registry.get_package, name)
            if success and not package:is_installed() then
                package:install()
            end

            local lsp = require("mason-lspconfig").get_mappings().package_to_lspconfig[name]
            config.capabilities = require("blink.cmp").get_lsp_capabilities()
            require("lspconfig")[lsp].setup(config)
        end

        -- 自定义 servers 列表，将要安装的 LSP 服务器和相关配置写在列表中。
        local servers = {
            -- lua 语言服务器
            ["lua-language-server"] = {
                -- lua-language-server 的优化配置，将 vim 定义为 lua 语言服务器可以识别的全局变量。
                settings = {
                    Lua = {
                        diagnostics = {
                            globals = { "vim" },
                        },
                    }
                }
            },
            -- python 语言服务器
            pyright = {},
            -- html 语言服务器
            ["html-lsp"] = {},
            -- css 语言服务器
            ["css-lsp"] = {},
            -- javascript 语言服务器
            ["typescript-language-server"] = {},
            -- emmmet 语言服务器
            ["emmet-ls"] = {},
        }

        -- 使用循环语句搭配自定义的 setup 函数安装列表中的语言服务器
        for server, config in pairs(servers) do
            setup(server, config)
        end

        vim.cmd("LspStart")
        vim.diagnostic.config({
            virtual_text = true,
            update_in_insert = true
        })
    end
}

```
