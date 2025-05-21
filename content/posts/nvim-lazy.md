+++
date = '2025-05-21T09:32:21+08:00'
draft = false
title = 'Nvim插件管理器——Lazy'
categories = ["nvim"]
tags = ["nvim", "lazy"]
+++
Neovim 好用的很大一部分原因是开源社区提供了大量的 nvim 插件。依靠这些插件，用户可以很自由的定制属于自己的 nvim 配置。如果用户引入的插件过多，会导致 nvim 的配置过程很复杂，同时也不具有移植性。
所以，主流的方式是通过nvim的插件管理器，对插件进行统一的安装和管理。目前（2025年）主流的 nvim 插件管理器就是 [Lazy.nvim](https://github.com/folke/lazy.nvim)。

---
# 插件管理器`Lazy`
`Lazy.nvim` 插件管理器由 Folke Lemaitre 在 GitHub 上发布。该作者还同时发布了需要厉害的插件。
## 安装
按照 Docs 中的操作说明，将配置文件放置到nvim的配置目录中。然后再进行调用。

比如：在 /nvim/lua/core/ 目录中新建 lazy.lua 文件。然后将 lazy 的配置代码粘贴到该文件中。最后，在 init.lua 文件中通过 `require("core.lazy")` 引用该插件。

## 配置文件 `lazy.lua`
```lua
-- Bootstrap lazy.nvim
-- 设置插件的安装目录，其中".."为lua语句中的连接字符
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
-- 判断lazy是否已经安装，没有安装的话就进行安装
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({ "git", "clone", "--filter=blob:none", "--branch=stable", lazyrepo, lazypath })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- Make sure to setup `mapleader` and `maplocalleader` before
-- loading lazy.nvim so that mappings are correct.
-- This is also a good place to setup other settings (vim.opt)
-- 设置 <leader> 键为空格
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Setup lazy.nvim
require("lazy").setup({
  spec = {
    -- import your plugins
    { import = "plugins" },
  },
  -- Configure any other settings here. See the documentation for more details.
  -- colorscheme that will be used when installing plugins.
  install = { colorscheme = { "habamax" } },
  -- automatically check for plugin updates
  checker = { enabled = true },
})

-- 默认下，是通过在 command-mode 下输入 `:Lazy<CR>` 启动 Lazy， <CR> 为回车。
-- 这里是设置了启动 Lazy 的快捷键
vim.keymap.set("n", "<leader>L", "<CMD>Lazy<CR>", {desc = "Open Lazy"})
```

---
# 插件安装：以 Tokyonight 为例
1. 在 Github 中查找 tokyonight.nvim, 并复制 lua 配置文件
2. 在 nvim/lua/plugins/ 文件夹中，新建 tokyonight.lua 文件。并将网页内容粘贴
> 注：复制下来的 lua 配置文件只有大括号及大括号中的内容。需要在最前方加入 `return` Lazy才能调用。
>```lua
>return{
>  -- 插件的 GitHub 地址，https://github.com/folke/tokyonight.nvim
>  -- https://github.com 部分可以不写，Lazy 会自动补全
>  "folke/tokyonight.nvim",
>  lazy = false,
>  priority = 1000,
>  -- opts 中的内容是针对该插件一些设置
>  opts = {},
>}
>```
3. 在 neovim 的 <command-mode> 中输入 `:colorscheme tokyonight<CR>` 即可变更主题。

---
# Lazy 优化配置
该模块的内容依然以 tokyonight 为例进行设置
## 让插件被加载的同时调用配置
当前插件的配置，需要每次进入都运行 `colorscheme` 命令才能实现。
如何才能让插件被加载的同时调用配置？
- Lazy 默认使用 `require("<plugin-name>").setup(opts)` 加载插件。我们可以通过添加函数的方式，在我们需要的时候调用插件。
    具体实现如下：
    ```lua
    return {
        "folke/tokyonight.nvim",
        lazy = false,
        opts = {
            style = "moon",
        },
        -- 添加一个 config 函数，该函数接受两个参数
        -- 第一个参数几乎毫无作用所以我们直接用 _ 替代
        -- 第二个参数则是我们前面的 opts
        -- 如无opts可直接写为 config = function()
        config = function (_, opts)
            require("tokyonight").setup(opts)
        end
    }
    ```

- 加载插件后通过 lua 的 `vim.cmd("<command>")` 命令执行主题切换即可。
    具体实现如下
    ```lua
    return {
        "folke/tokyonight.nvim",
        opts = {
            style = "day",
        },
        config = function (_, opts)
            require("tokyonight").setup(opts)
            vim.cmd("colorscheme tokyonight")
        end
    }
    ```
## require().setup() 方法的进一步说明
Lazy 只会对设置了 opts 或 function 属性的插件执行 `require("<plugins.name>").setup(opts)` 方法。但并不是所有的插件都需要调用 `require("<plugins.name>").setup(opts)` 方法
- tokyonight 这样的插件，只需正常加载即可生效。function方法是为了实现加载自动调用才进行添加的，即使没有 opts 和 function() 也不影响 tokyonight 插件的使用。
- bufferline 这样的插件，加载后需要调用 setup() 方法才可以生效。

## Lazy 的懒加载(lazy loading)
懒加载 (lazy loading)，是 Lazy 对插件加载的一种优化行为，即仅在需要的时候加载插件。
默认情况下，lazy 会在启动 neovim 的时候就加载所有插件，加载完成后才会进入 neovim 的界面。若插件过多，就会导致加载时间过长。但其实，我们并不是一开始就需要加载所有的插件。懒加载就是按需加载插件的一种优化。

lazy 中实现这个功能，可以通过以下几个属性：
- event：在某个事件触发的时候加载插件
    - 若插件想进行懒加载，但是却不知道什么时候进行懒加载时，可以将设置 event = "VeryLazy"。
- cmd：在某个命令被执行的时候加载插件
- ft：当前 buffer 为特定文件类型的时候加载插件
- keys：当触发快捷键时加载插件，如果快捷键不存在则创建快捷键

若不需要进行懒加载，则在配置文件中写入 `lazy = false`

## 声明依赖项--dependencies
当某个插件需要依赖另外一些插件。lazy 允许我们在安装插件的同时自动安装其依赖项。
具体方法为在配置文件中加入 dependencies 属性。

