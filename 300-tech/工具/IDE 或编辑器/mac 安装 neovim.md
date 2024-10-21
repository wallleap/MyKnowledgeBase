---
title: mac 安装 neovim
date: 2024-08-30T10:27:26+08:00
updated: 2024-09-02T01:44:31+08:00
dg-publish: false
source: https://www.josean.com/posts/how-to-setup-neovim-2024
---

除了 neovim，[spacevim](https://spacevim.org/cn/) 也是个不错的选择

安装字体

```sh
$ brew install font-meslo-lg-nerd-font
```

之后在 iTerm2 中修改字体为 **MesloLGS Nerd Font Mono**

安装 neovim

```sh
$ brew install neovim
$ nvim --version
```

安装

```sh
$ brew install ripgrep node
```

直接使用别人的配置文件

```sh
$ git clone git@github.com:josean-dev/dev-environment-files.git
$ cd dev-environment-files
$ cp -r .config/nvim ~/.config/nvim
$ nvim
```

## 目录和文件概述

- **`~/.config/nvim/`**
	- 这是 Neovim 的主配置目录。

### `after/`

- **作用**：包含在主 Neovim 配置 (`init.lua`) 加载之后的文件。这对于覆盖或扩展配置非常有用。
- **`queries/ecma/textobjects.scm`**：很可能包含 ECMAScript（JavaScript/TypeScript）的文本对象查询定义。

### `init.lua`

- **作用**：这是 Neovim 的主要配置文件，在 Neovim 启动时执行，通常用于设置整体配置、加载插件和应用设置。

### `lazy-lock.json`

- **作用**：由 `lazy.nvim` 插件管理器生成的文件，用于锁定插件的版本，确保配置的一致性。

### `lua/`

- 这个目录用于存储 Neovim 的 Lua 模块和配置。
- **`josean/`**
	- **作用**：这是 `lua/` 目录下的自定义目录，用于组织个人的 Neovim Lua 配置。
	- **`core/`**
		- **`init.lua`**：初始化 Neovim 的核心配置或设置。
		- **`keymaps.lua`**：定义自定义的键位映射。
		- **`options.lua`**：定义各种 Neovim 选项和设置。
	- **`lazy.lua`**：配置 `lazy.nvim` 插件管理器，用于管理和加载插件。
	- **`plugins/`**
		- **作用**：包含单独的 Lua 文件，用于配置特定的 Neovim 插件。
		- **`alpha.lua`**：配置 `alpha.nvim` 插件，通常用于创建启动屏幕或仪表板。
		- **`auto-session.lua`**：配置 `auto-session.nvim` 插件，用于自动保存和恢复 Neovim 会话。
		- **`autopairs.lua`**：配置 `nvim-autopairs` 插件，用于自动配对括号和引号。
		- **`bufferline.lua`**：配置 `bufferline.nvim` 插件，用于管理和显示缓冲区标签。
		- **`colorscheme.lua`**：设置 Neovim 的颜色主题。
		- **`comment.lua`**：配置 `Comment.nvim` 插件，用于轻松注释代码。
		- **`dressing.lua`**：配置 `dressing.nvim` 插件，用于增强 Neovim 的用户界面组件，如提示和输入对话框。
		- **`formatting.lua`**：配置格式化插件或设置。
		- **`gitsigns.lua`**：配置 `gitsigns.nvim` 插件，用于在边栏显示 git 状态标记。
		- **`indent-blankline.lua`**：配置 `indent-blankline.nvim` 插件，为代码添加缩进指示线。
		- **`init.lua`**：可能用于初始化其他配置或作为插件配置的入口点。
		- **`lazygit.lua`**：配置 `lazygit.nvim` 插件，将 `lazygit` 终端应用程序集成到 Neovim 中。
		- **`linting.lua`**：配置 linting 插件或设置。
		- **`lsp/`**

				- **`lspconfig.lua`**：配置 `nvim-lspconfig` 插件，用于配置 Neovim 内置的 LSP（语言服务器协议）客户端。
				- **`mason.lua`**：配置 `mason.nvim` 插件，用于管理外部工具和 LSP 服务器。
		- **`lualine.lua`**：配置 `lualine.nvim` 插件，用于 Neovim 的状态栏。
		- **`nvim-cmp.lua`**：配置 `nvim-cmp` 插件，用于补全功能。
		- **`nvim-tree.lua`**：配置 `nvim-tree.lua` 插件，用于文件浏览器。
		- **`nvim-treesitter-text-objects.lua`**：配置 `nvim-treesitter` 插件，特别是 Treesitter 提供的文本对象。
		- **`substitute.lua`**：配置替代或查找替换插件。
		- **`surround.lua`**：配置 `nvim-surround` 插件，用于处理括号、引号等包围字符。
		- **`telescope.lua`**：配置 `telescope.nvim` 插件，一个模糊查找插件，用于搜索和浏览文件。
		- **`todo-comments.lua`**：配置 `todo-comments.nvim` 插件，用于高亮 TODO 注释。
		- **`treesitter.lua`**：配置 `nvim-treesitter` 插件，提供高级语法高亮和代码分析。
		- **`trouble.lua`**：配置 `trouble.nvim` 插件，用于更友好的查看和导航诊断信息。
		- **`vim-maximizer.lua`**：配置 `vim-maximizer.nvim` 插件，用于最大化和恢复窗口。
		- **`which-key.lua`**：配置 `which-key.nvim` 插件，用于显示可用的按键绑定。

这种结构提供了模块化和有组织的 Neovim 配置方式，使管理和维护配置变得更容易。

You can find the source code for my config [here](https://github.com/josean-dev/dev-environment-files).

## Open a terminal window

Open a terminal window on your mac. You will need a true color terminal for the colorscheme to work properly.

I’m using *iTerm2*

## Install Homebrew

Run the following command:

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

If necessary, when prompted, enter your password here and press enter. If you haven’t installed the XCode Command Line Tools, when prompted, press enter and homebrew will install this as well.

## Add To Path (Only Apple Silicon Macbooks)

After installing, add it to the path. This step shouldn’t be necessary on Intel macs.

Run the following two commands to do so:

```
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## Install iTerm2 If Necessary

If you don’t have a true color terminal, install iTerm2 with homebrew:

```
brew install --cask iterm2
```

Then switch to this terminal.

## Install A Nerd Font

I use Meslo Nerd Font. To install it do:

```
brew tap homebrew/cask-fonts
```

And then do:

```
brew install font-meslo-lg-nerd-font
```

Then open iTerm2 settings with `CMD+,` and under **Profiles > Text** change the font to **MesloLGS Nerd Font Mono**

## Install Neovim

Run:

```
brew install neovim
```

## Install Ripgrep

Run:

```
brew install ripgrep
```

## Install Node

Run:

```
brew install node
```

## Setup Initial File Structure

Your config will be located in `~/.config/nvim`.

Let’s setup the initial file structure with the following commands:

Make the nvim config directory.

```
mkdir -p ~/.config/nvim
```

*`-p` is used to also create parent directories if they don’t already exist*

Move to this directory:

```
cd ~/.config/nvim
```

Create main `init.lua` file:

```
touch init.lua
```

Create `lua/josean/core` directories:

*Any time I use “josean” you can replace this with your name*

```
mkdir -p lua/josean/core
```

Create plugins directory (will have all of the plugin configs/specs):

```
mkdir -p lua/josean/plugins
```

Create `lazy.lua` file (will be used to setup/configure lazy.nvim plugin manager):

```
touch lua/josean/lazy.lua
```

## Setup core options

Make sure you’re in `~/.config/nvim` and open the config:

```
nvim .
```

Navigate to the core folder and press `%` to create a file and call it: “options.lua”

In this file add:

```
vim.cmd("let g:netrw_liststyle = 3")
```

Open the explorer with `:Explore` and navigate to the main `init.lua` file.

Add the following to load the basic options on startup:

```
require("josean.core.options")
```

Close Neovim with `:w` and reopen it with `nvim .`

Go back to “options.lua” and add the following to setup the rest of the options:

```
local opt = vim.opt -- for conciseness

-- line numbers
opt.relativenumber = true -- show relative line numbers
opt.number = true -- shows absolute line number on cursor line (when relative number is on)

-- tabs & indentation
opt.tabstop = 2 -- 2 spaces for tabs (prettier default)
opt.shiftwidth = 2 -- 2 spaces for indent width
opt.expandtab = true -- expand tab to spaces
opt.autoindent = true -- copy indent from current line when starting new one

-- line wrapping
opt.wrap = false -- disable line wrapping

-- search settings
opt.ignorecase = true -- ignore case when searching
opt.smartcase = true -- if you include mixed case in your search, assumes you want case-sensitive

-- cursor line
opt.cursorline = true -- highlight the current cursor line

-- appearance

-- turn on termguicolors for nightfly colorscheme to work
-- (have to use iterm2 or any other true color terminal)
opt.termguicolors = true
opt.background = "dark" -- colorschemes that can be light or dark will be made dark
opt.signcolumn = "yes" -- show sign column so that text doesn't shift

-- backspace
opt.backspace = "indent,eol,start" -- allow backspace on indent, end of line or insert mode start position

-- clipboard
opt.clipboard:append("unnamedplus") -- use system clipboard as default register

-- split windows
opt.splitright = true -- split vertical window to the right
opt.splitbelow = true -- split horizontal window to the bottom

-- turn off swapfile
opt.swapfile = false
```

Do `:e lua/josean/core/init.lua`

Add the following:

```
require("josean.core.options")
```

Open the explorer with `:Explore` and go to the main init.lua file and change the require to:

```
require("josean.core")
```

## Setup core keymaps

Do `:e lua/josean/core/keymaps.lua`

And add the following to this file:

```
-- set leader key to space
vim.g.mapleader = " "

local keymap = vim.keymap -- for conciseness

---------------------
-- General Keymaps -------------------

-- use jk to exit insert mode
keymap.set("i", "jk", "<ESC>", { desc = "Exit insert mode with jk" })

-- clear search highlights
keymap.set("n", "<leader>nh", ":nohl<CR>", { desc = "Clear search highlights" })

-- delete single character without copying into register
-- keymap.set("n", "x", '"_x')

-- increment/decrement numbers
keymap.set("n", "<leader>+", "<C-a>", { desc = "Increment number" }) -- increment
keymap.set("n", "<leader>-", "<C-x>", { desc = "Decrement number" }) -- decrement

-- window management
keymap.set("n", "<leader>sv", "<C-w>v", { desc = "Split window vertically" }) -- split window vertically
keymap.set("n", "<leader>sh", "<C-w>s", { desc = "Split window horizontally" }) -- split window horizontally
keymap.set("n", "<leader>se", "<C-w>=", { desc = "Make splits equal size" }) -- make split windows equal width & height
keymap.set("n", "<leader>sx", "<cmd>close<CR>", { desc = "Close current split" }) -- close current split window

keymap.set("n", "<leader>to", "<cmd>tabnew<CR>", { desc = "Open new tab" }) -- open new tab
keymap.set("n", "<leader>tx", "<cmd>tabclose<CR>", { desc = "Close current tab" }) -- close current tab
keymap.set("n", "<leader>tn", "<cmd>tabn<CR>", { desc = "Go to next tab" }) --  go to next tab
keymap.set("n", "<leader>tp", "<cmd>tabp<CR>", { desc = "Go to previous tab" }) --  go to previous tab
keymap.set("n", "<leader>tf", "<cmd>tabnew %<CR>", { desc = "Open current buffer in new tab" }) --  move current buffer to new tab
```

Open the explorer with `:Explore`, open `lua/josean/core/init.lua` and add the following:

```
require("josean.core.options")
require("josean.core.keymaps")
```

Exit with `:q` and reenter Neovim with `nvim .`

## Setup lazy.nvim

Go to “lazy.lua” and add the following to bootstrap lazy.nvim

```
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```

Then configure lazy.nvim with the following:

```
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup("josean.plugins")
```

*If you’re using your name instead of “josean”, change that to your name here as well*

Then open the explorer with `:Explore` and navigate to main `init.lua` file.

Add the following to it:

```
require("josean.core")
require("josean.lazy")
```

Exit with `:q` and reenter Neovim with `nvim`

**You can see the lazy.nvim UI now with `:Lazy` and you can close the UI with `q`**

## Install plenary & vim-tmux-navigator

Do `:e lua/josean/plugins/init.lua`

Add the following to install **plenary** and **vim-tmux-navigator**:

```
return {
  "nvim-lua/plenary.nvim", -- lua functions that many plugins use
  "christoomey/vim-tmux-navigator", -- tmux & split window navigation
}
```

After adding this, save the file and you can install manually by doing `:Lazy`, then typing `I`.

After install, close the UI with `q` and you can manually load a plugin with `:Lazy reload vim-tmux-navigator` for example.

Otherwise, you can also exit with `:q` and reenter Neovim with `nvim .` and it’ll happen automatically.

## Install & configure tokyonight colorscheme

Do `:e lua/josean/plugins/colorscheme.lua`

In this file add the following:

```
return {
  {
    "folke/tokyonight.nvim",
    priority = 1000, -- make sure to load this before all the other start plugins
    config = function()
      local bg = "#011628"
      local bg_dark = "#011423"
      local bg_highlight = "#143652"
      local bg_search = "#0A64AC"
      local bg_visual = "#275378"
      local fg = "#CBE0F0"
      local fg_dark = "#B4D0E9"
      local fg_gutter = "#627E97"
      local border = "#547998"

      require("tokyonight").setup({
        style = "night",
        on_colors = function(colors)
          colors.bg = bg
          colors.bg_dark = bg_dark
          colors.bg_float = bg_dark
          colors.bg_highlight = bg_highlight
          colors.bg_popup = bg_dark
          colors.bg_search = bg_search
          colors.bg_sidebar = bg_dark
          colors.bg_statusline = bg_dark
          colors.bg_visual = bg_visual
          colors.border = border
          colors.fg = fg
          colors.fg_dark = fg_dark
          colors.fg_float = fg
          colors.fg_gutter = fg_gutter
          colors.fg_sidebar = fg_dark
        end,
      })
      -- load the colorscheme here
      vim.cmd([[colorscheme tokyonight]])
    end,
  },
}
```

This will setup **tokyonight** as the colorscheme and modify some of its colors according to my preference.

Exit with `:q` and reenter Neovim with `nvim .`

## Setup nvim-tree file explorer

Do `:e lua/josean/plugins/nvim-tree.lua`

Add the following to this file:

```
return {
  "nvim-tree/nvim-tree.lua",
  dependencies = "nvim-tree/nvim-web-devicons",
  config = function()
    local nvimtree = require("nvim-tree")

    -- recommended settings from nvim-tree documentation
    vim.g.loaded_netrw = 1
    vim.g.loaded_netrwPlugin = 1

    nvimtree.setup({
      view = {
        width = 35,
        relativenumber = true,
      },
      -- change folder arrow icons
      renderer = {
        indent_markers = {
          enable = true,
        },
        icons = {
          glyphs = {
            folder = {
              arrow_closed = "", -- arrow when folder is closed
              arrow_open = "", -- arrow when folder is open
            },
          },
        },
      },
      -- disable window_picker for
      -- explorer to work well with
      -- window splits
      actions = {
        open_file = {
          window_picker = {
            enable = false,
          },
        },
      },
      filters = {
        custom = { ".DS_Store" },
      },
      git = {
        ignore = false,
      },
    })

    -- set keymaps
    local keymap = vim.keymap -- for conciseness

    keymap.set("n", "<leader>ee", "<cmd>NvimTreeToggle<CR>", { desc = "Toggle file explorer" }) -- toggle file explorer
    keymap.set("n", "<leader>ef", "<cmd>NvimTreeFindFileToggle<CR>", { desc = "Toggle file explorer on current file" }) -- toggle file explorer on current file
    keymap.set("n", "<leader>ec", "<cmd>NvimTreeCollapse<CR>", { desc = "Collapse file explorer" }) -- collapse file explorer
    keymap.set("n", "<leader>er", "<cmd>NvimTreeRefresh<CR>", { desc = "Refresh file explorer" }) -- refresh file explorer
  end
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup which-key

Which-key is great for seeing what keymaps you can use.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `which-key.lua`

Add this to the file:

```
return {
  "folke/which-key.nvim",
  event = "VeryLazy",
  init = function()
    vim.o.timeout = true
    vim.o.timeoutlen = 500
  end,
  opts = {
    -- your configuration comes here
    -- or leave it empty to use the default settings
    -- refer to the configuration section below
  },
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup telescope fuzzy finder

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `telescope.lua`

Add this to the file:

```
return {
  "nvim-telescope/telescope.nvim",
  branch = "0.1.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    { "nvim-telescope/telescope-fzf-native.nvim", build = "make" },
    "nvim-tree/nvim-web-devicons",
  },
  config = function()
    local telescope = require("telescope")
    local actions = require("telescope.actions")

    telescope.setup({
      defaults = {
        path_display = { "smart" },
        mappings = {
          i = {
            ["<C-k>"] = actions.move_selection_previous, -- move to prev result
            ["<C-j>"] = actions.move_selection_next, -- move to next result
            ["<C-q>"] = actions.send_selected_to_qflist + actions.open_qflist,
          },
        },
      },
    })

    telescope.load_extension("fzf")

    -- set keymaps
    local keymap = vim.keymap -- for conciseness

    keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<cr>", { desc = "Fuzzy find files in cwd" })
    keymap.set("n", "<leader>fr", "<cmd>Telescope oldfiles<cr>", { desc = "Fuzzy find recent files" })
    keymap.set("n", "<leader>fs", "<cmd>Telescope live_grep<cr>", { desc = "Find string in cwd" })
    keymap.set("n", "<leader>fc", "<cmd>Telescope grep_string<cr>", { desc = "Find string under cursor in cwd" })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup a greeter

We’re gonna setup a greeter for Neovim startup with alpha-nvim

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `alpha.lua`

Add the following code:

```
return {
  "goolord/alpha-nvim",
  event = "VimEnter",
  config = function()
    local alpha = require("alpha")
    local dashboard = require("alpha.themes.dashboard")

    -- Set header
    dashboard.section.header.val = {
      "                                                     ",
      "  ███╗   ██╗███████╗ ██████╗ ██╗   ██╗██╗███╗   ███╗ ",
      "  ████╗  ██║██╔════╝██╔═══██╗██║   ██║██║████╗ ████║ ",
      "  ██╔██╗ ██║█████╗  ██║   ██║██║   ██║██║██╔████╔██║ ",
      "  ██║╚██╗██║██╔══╝  ██║   ██║╚██╗ ██╔╝██║██║╚██╔╝██║ ",
      "  ██║ ╚████║███████╗╚██████╔╝ ╚████╔╝ ██║██║ ╚═╝ ██║ ",
      "  ╚═╝  ╚═══╝╚══════╝ ╚═════╝   ╚═══╝  ╚═╝╚═╝     ╚═╝ ",
      "                                                     ",
    }

    -- Set menu
    dashboard.section.buttons.val = {
      dashboard.button("e", "  > New File", "<cmd>ene<CR>"),
      dashboard.button("SPC ee", "  > Toggle file explorer", "<cmd>NvimTreeToggle<CR>"),
      dashboard.button("SPC ff", "󰱼 > Find File", "<cmd>Telescope find_files<CR>"),
      dashboard.button("SPC fs", "  > Find Word", "<cmd>Telescope live_grep<CR>"),
      dashboard.button("SPC wr", "󰁯  > Restore Session For Current Directory", "<cmd>SessionRestore<CR>"),
      dashboard.button("q", " > Quit NVIM", "<cmd>qa<CR>"),
    }

    -- Send config to alpha
    alpha.setup(dashboard.opts)

    -- Disable folding on alpha buffer
    vim.cmd([[autocmd FileType alpha setlocal nofoldenable]])
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup automated session manager

Automatic session management is great for auto saving sessions before exiting Neovim and getting back to work when you come back.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `auto-session.lua`

Add the following to this file:

```
return {
  "rmagatti/auto-session",
  config = function()
    local auto_session = require("auto-session")

    auto_session.setup({
      auto_restore_enabled = false,
      auto_session_suppress_dirs = { "~/", "~/Dev/", "~/Downloads", "~/Documents", "~/Desktop/" },
    })

    local keymap = vim.keymap

    keymap.set("n", "<leader>wr", "<cmd>SessionRestore<CR>", { desc = "Restore session for cwd" }) -- restore last workspace session for current directory
    keymap.set("n", "<leader>ws", "<cmd>SessionSave<CR>", { desc = "Save session for auto session root dir" }) -- save workspace session for current working directory
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim .`

When working in a project, you can now close everything with `:qa` and when you open Neovim again in this directory you can use `<leader>wr` to restore your workspace/session.

## Disable lazy.nvim change_detection notification

Let’s disable the lazy.nvim change_detection notification which I find a bit annoying.

Navigate to `lazy.lua` and modify the code like so:

```
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup("josean.plugins", {
  change_detection = {
    notify = false,
  },
})
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup bufferline for better looking tabs

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `bufferline.lua`

Add the following code:

```
return {
  "akinsho/bufferline.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  version = "*",
  opts = {
    options = {
      mode = "tabs",
      separator_style = "slant",
    },
  },
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup lualine for a better statusline

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `lualine.lua`

Add the following code:

```
return {
  "nvim-lualine/lualine.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons" },
  config = function()
    local lualine = require("lualine")
    local lazy_status = require("lazy.status") -- to configure lazy pending updates count

    local colors = {
      blue = "#65D1FF",
      green = "#3EFFDC",
      violet = "#FF61EF",
      yellow = "#FFDA7B",
      red = "#FF4A4A",
      fg = "#c3ccdc",
      bg = "#112638",
      inactive_bg = "#2c3043",
    }

    local my_lualine_theme = {
      normal = {
        a = { bg = colors.blue, fg = colors.bg, gui = "bold" },
        b = { bg = colors.bg, fg = colors.fg },
        c = { bg = colors.bg, fg = colors.fg },
      },
      insert = {
        a = { bg = colors.green, fg = colors.bg, gui = "bold" },
        b = { bg = colors.bg, fg = colors.fg },
        c = { bg = colors.bg, fg = colors.fg },
      },
      visual = {
        a = { bg = colors.violet, fg = colors.bg, gui = "bold" },
        b = { bg = colors.bg, fg = colors.fg },
        c = { bg = colors.bg, fg = colors.fg },
      },
      command = {
        a = { bg = colors.yellow, fg = colors.bg, gui = "bold" },
        b = { bg = colors.bg, fg = colors.fg },
        c = { bg = colors.bg, fg = colors.fg },
      },
      replace = {
        a = { bg = colors.red, fg = colors.bg, gui = "bold" },
        b = { bg = colors.bg, fg = colors.fg },
        c = { bg = colors.bg, fg = colors.fg },
      },
      inactive = {
        a = { bg = colors.inactive_bg, fg = colors.semilightgray, gui = "bold" },
        b = { bg = colors.inactive_bg, fg = colors.semilightgray },
        c = { bg = colors.inactive_bg, fg = colors.semilightgray },
      },
    }

    -- configure lualine with modified theme
    lualine.setup({
      options = {
        theme = my_lualine_theme,
      },
      sections = {
        lualine_x = {
          {
            lazy_status.updates,
            cond = lazy_status.has_updates,
            color = { fg = "#ff9e64" },
          },
          { "encoding" },
          { "fileformat" },
          { "filetype" },
        },
      },
    })
  end,
}
```

So that lualine can show pending plugin updates through lazy.nvim, open “lazy.lua” and modify it like so:

```
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup("josean.plugins", {
  checker = {
    enabled = true,
    notify = false,
  },
  change_detection = {
    notify = false,
  },
})
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup dressing.nvim

Dressing.nvim improves the ui for `vim.ui.select` and `vim.ui.input`

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `dressing.lua`

Add the following code:

```
return {
  "stevearc/dressing.nvim",
  event = "VeryLazy",
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup vim-maximizer

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `vim-maximizer.lua`

Add the following code:

```
return {
  "szw/vim-maximizer",
  keys = {
    { "<leader>sm", "<cmd>MaximizerToggle<CR>", desc = "Maximize/minimize a split" },
  },
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup treesitter

Treesitter is an awesome Neovim feature that provides better syntax highlighting, indentation, autotagging, incremental selection and many other cool features.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `treesitter.lua`

Add the following code:

```
return {
  "nvim-treesitter/nvim-treesitter",
  event = { "BufReadPre", "BufNewFile" },
  build = ":TSUpdate",
  dependencies = {
    "windwp/nvim-ts-autotag",
  },
  config = function()
    -- import nvim-treesitter plugin
    local treesitter = require("nvim-treesitter.configs")

    -- configure treesitter
    treesitter.setup({ -- enable syntax highlighting
      highlight = {
        enable = true,
      },
      -- enable indentation
      indent = { enable = true },
      -- enable autotagging (w/ nvim-ts-autotag plugin)
      autotag = {
        enable = true,
      },
      -- ensure these language parsers are installed
      ensure_installed = {
        "json",
        "javascript",
        "typescript",
        "tsx",
        "yaml",
        "html",
        "css",
        "prisma",
        "markdown",
        "markdown_inline",
        "svelte",
        "graphql",
        "bash",
        "lua",
        "vim",
        "dockerfile",
        "gitignore",
        "query",
        "vimdoc",
        "c",
      },
      incremental_selection = {
        enable = true,
        keymaps = {
          init_selection = "<C-space>",
          node_incremental = "<C-space>",
          scope_incremental = false,
          node_decremental = "<bs>",
        },
      },
    })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup indent guides

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `indent-blankline.lua`

Add the following code:

```
return {
  "lukas-reineke/indent-blankline.nvim",
  event = { "BufReadPre", "BufNewFile" },
  main = "ibl",
  opts = {
    indent = { char = "┊" },
  },
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup autocompletion

We’re going to setup completion with “nvim-cmp” to get suggestions as we type.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `nvim-cmp.lua`

Add the following code:

```
return {
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    "hrsh7th/cmp-buffer", -- source for text in buffer
    "hrsh7th/cmp-path", -- source for file system paths
    {
      "L3MON4D3/LuaSnip",
      -- follow latest release.
      version = "v2.*", -- Replace <CurrentMajor> by the latest released major (first number of latest release)
      -- install jsregexp (optional!).
      build = "make install_jsregexp",
    },
    "saadparwaiz1/cmp_luasnip", -- for autocompletion
    "rafamadriz/friendly-snippets", -- useful snippets
    "onsails/lspkind.nvim", -- vs-code like pictograms
  },
  config = function()
    local cmp = require("cmp")

    local luasnip = require("luasnip")

    local lspkind = require("lspkind")

    -- loads vscode style snippets from installed plugins (e.g. friendly-snippets)
    require("luasnip.loaders.from_vscode").lazy_load()

    cmp.setup({
      completion = {
        completeopt = "menu,menuone,preview,noselect",
      },
      snippet = { -- configure how nvim-cmp interacts with snippet engine
        expand = function(args)
          luasnip.lsp_expand(args.body)
        end,
      },
      mapping = cmp.mapping.preset.insert({
        ["<C-k>"] = cmp.mapping.select_prev_item(), -- previous suggestion
        ["<C-j>"] = cmp.mapping.select_next_item(), -- next suggestion
        ["<C-b>"] = cmp.mapping.scroll_docs(-4),
        ["<C-f>"] = cmp.mapping.scroll_docs(4),
        ["<C-Space>"] = cmp.mapping.complete(), -- show completion suggestions
        ["<C-e>"] = cmp.mapping.abort(), -- close completion window
        ["<CR>"] = cmp.mapping.confirm({ select = false }),
      }),
      -- sources for autocompletion
      sources = cmp.config.sources({
        { name = "luasnip" }, -- snippets
        { name = "buffer" }, -- text within current buffer
        { name = "path" }, -- file system paths
      }),

      -- configure lspkind for vs-code like pictograms in completion menu
      formatting = {
        format = lspkind.cmp_format({
          maxwidth = 50,
          ellipsis_char = "...",
        }),
      },
    })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup auto closing pairs

This plugin will help us auto close surrounding characters like parens, brackets, curly braces, quotes, single quotes and tags

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `autopairs.lua`

Add the following code:

```
return {
  "windwp/nvim-autopairs",
  event = { "InsertEnter" },
  dependencies = {
    "hrsh7th/nvim-cmp",
  },
  config = function()
    -- import nvim-autopairs
    local autopairs = require("nvim-autopairs")

    -- configure autopairs
    autopairs.setup({
      check_ts = true, -- enable treesitter
      ts_config = {
        lua = { "string" }, -- don't add pairs in lua string treesitter nodes
        javascript = { "template_string" }, -- don't add pairs in javscript template_string treesitter nodes
        java = false, -- don't check treesitter on java
      },
    })

    -- import nvim-autopairs completion functionality
    local cmp_autopairs = require("nvim-autopairs.completion.cmp")

    -- import nvim-cmp plugin (completions plugin)
    local cmp = require("cmp")

    -- make autopairs and completion work together
    cmp.event:on("confirm_done", cmp_autopairs.on_confirm_done())
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup commenting plugin

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `comment.lua`

Add the following code:

```
return {
  "numToStr/Comment.nvim",
  event = { "BufReadPre", "BufNewFile" },
  dependencies = {
    "JoosepAlviste/nvim-ts-context-commentstring",
  },
  config = function()
    -- import comment plugin safely
    local comment = require("Comment")

    local ts_context_commentstring = require("ts_context_commentstring.integrations.comment_nvim")

    -- enable comment
    comment.setup({
      -- for commenting tsx, jsx, svelte, html files
      pre_hook = ts_context_commentstring.create_pre_hook(),
    })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup todo comments

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `todo-comments.lua`

Add the following code:

```
return {
  "folke/todo-comments.nvim",
  event = { "BufReadPre", "BufNewFile" },
  dependencies = { "nvim-lua/plenary.nvim" },
  config = function()
    local todo_comments = require("todo-comments")

    -- set keymaps
    local keymap = vim.keymap -- for conciseness

    keymap.set("n", "]t", function()
      todo_comments.jump_next()
    end, { desc = "Next todo comment" })

    keymap.set("n", "[t", function()
      todo_comments.jump_prev()
    end, { desc = "Previous todo comment" })

    todo_comments.setup()
  end,
}
```

Look for `telescope.lua` with telescope with `<leader>ff`

Open this file and add the following to be able to look for todos with telescope:

```
return {
  "nvim-telescope/telescope.nvim",
  branch = "0.1.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    { "nvim-telescope/telescope-fzf-native.nvim", build = "make" },
    "nvim-tree/nvim-web-devicons",
    "folke/todo-comments.nvim",
  },
  config = function()
    local telescope = require("telescope")
    local actions = require("telescope.actions")

    telescope.setup({
      defaults = {
        path_display = { "smart" },
        mappings = {
          i = {
            ["<C-k>"] = actions.move_selection_previous, -- move to prev result
            ["<C-j>"] = actions.move_selection_next, -- move to next result
            ["<C-q>"] = actions.send_selected_to_qflist + actions.open_qflist,
          },
        },
      },
    })

    telescope.load_extension("fzf")

    -- set keymaps
    local keymap = vim.keymap -- for conciseness

    keymap.set("n", "<leader>ff", "<cmd>Telescope find_files<cr>", { desc = "Fuzzy find files in cwd" })
    keymap.set("n", "<leader>fr", "<cmd>Telescope oldfiles<cr>", { desc = "Fuzzy find recent files" })
    keymap.set("n", "<leader>fs", "<cmd>Telescope live_grep<cr>", { desc = "Find string in cwd" })
    keymap.set("n", "<leader>fc", "<cmd>Telescope grep_string<cr>", { desc = "Find string under cursor in cwd" })
    keymap.set("n", "<leader>ft", "<cmd>TodoTelescope<cr>", { desc = "Find todos" })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup substitution plugin

This plugin allows us to use `s` followed by a `motion` to substitute text that was previously copied.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `substitute.lua`

Add the following code:

```
return {
  "gbprod/substitute.nvim",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    local substitute = require("substitute")

    substitute.setup()

    -- set keymaps
    local keymap = vim.keymap -- for conciseness

    vim.keymap.set("n", "s", substitute.operator, { desc = "Substitute with motion" })
    vim.keymap.set("n", "ss", substitute.line, { desc = "Substitute line" })
    vim.keymap.set("n", "S", substitute.eol, { desc = "Substitute to end of line" })
    vim.keymap.set("x", "s", substitute.visual, { desc = "Substitute in visual mode" })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup nvim-surround

This plugin is great for adding, deleting and modifying surrounding symbols and tags.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `surround.lua`

Add the following code:

```
return {
  "kylechui/nvim-surround",
  event = { "BufReadPre", "BufNewFile" },
  version = "*", -- Use for stability; omit to use `main` branch for the latest features
  config = true,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup LSP

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `lua/josean/plugins` add a new directory with `a`, calling it `lsp/`

Navigate to `lazy.lua` and modify it so that `lazy.nvim` knows about the new `lsp` directory like so:

```
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({ { import = "josean.plugins" }, { import = "josean.plugins.lsp" } }, {
  checker = {
    enabled = true,
    notify = false,
  },
  change_detection = {
    notify = false,
  },
})
```

### Setup mason.nvim

Mason.nvim is used to install and manage all of the language servers that you need for the languages you work for.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins/lsp` add a new file with `a` and call it `mason.lua`

Add the following code:

```
return {
  "williamboman/mason.nvim",
  dependencies = {
    "williamboman/mason-lspconfig.nvim",
  },
  config = function()
    -- import mason
    local mason = require("mason")

    -- import mason-lspconfig
    local mason_lspconfig = require("mason-lspconfig")

    -- enable mason and configure icons
    mason.setup({
      ui = {
        icons = {
          package_installed = "✓",
          package_pending = "➜",
          package_uninstalled = "✗",
        },
      },
    })

    mason_lspconfig.setup({
      -- list of servers for mason to install
      ensure_installed = {
        "tsserver",
        "html",
        "cssls",
        "tailwindcss",
        "svelte",
        "lua_ls",
        "graphql",
        "emmet_ls",
        "prismals",
        "pyright",
      },
    })
  end,
}
```

### Setup nvim-lspconfig

Nvim-lspconfig is used to configure your language servers.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins/lsp` add a new file with `a` and call it `lspconfig.lua`

Add the following code:

```
return {
  "neovim/nvim-lspconfig",
  event = { "BufReadPre", "BufNewFile" },
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",
    { "antosha417/nvim-lsp-file-operations", config = true },
    { "folke/neodev.nvim", opts = {} },
  },
  config = function()
    -- import lspconfig plugin
    local lspconfig = require("lspconfig")

    -- import mason_lspconfig plugin
    local mason_lspconfig = require("mason-lspconfig")

    -- import cmp-nvim-lsp plugin
    local cmp_nvim_lsp = require("cmp_nvim_lsp")

    local keymap = vim.keymap -- for conciseness

    vim.api.nvim_create_autocmd("LspAttach", {
      group = vim.api.nvim_create_augroup("UserLspConfig", {}),
      callback = function(ev)
        -- Buffer local mappings.
        -- See `:help vim.lsp.*` for documentation on any of the below functions
        local opts = { buffer = ev.buf, silent = true }

        -- set keybinds
        opts.desc = "Show LSP references"
        keymap.set("n", "gR", "<cmd>Telescope lsp_references<CR>", opts) -- show definition, references

        opts.desc = "Go to declaration"
        keymap.set("n", "gD", vim.lsp.buf.declaration, opts) -- go to declaration

        opts.desc = "Show LSP definitions"
        keymap.set("n", "gd", "<cmd>Telescope lsp_definitions<CR>", opts) -- show lsp definitions

        opts.desc = "Show LSP implementations"
        keymap.set("n", "gi", "<cmd>Telescope lsp_implementations<CR>", opts) -- show lsp implementations

        opts.desc = "Show LSP type definitions"
        keymap.set("n", "gt", "<cmd>Telescope lsp_type_definitions<CR>", opts) -- show lsp type definitions

        opts.desc = "See available code actions"
        keymap.set({ "n", "v" }, "<leader>ca", vim.lsp.buf.code_action, opts) -- see available code actions, in visual mode will apply to selection

        opts.desc = "Smart rename"
        keymap.set("n", "<leader>rn", vim.lsp.buf.rename, opts) -- smart rename

        opts.desc = "Show buffer diagnostics"
        keymap.set("n", "<leader>D", "<cmd>Telescope diagnostics bufnr=0<CR>", opts) -- show  diagnostics for file

        opts.desc = "Show line diagnostics"
        keymap.set("n", "<leader>d", vim.diagnostic.open_float, opts) -- show diagnostics for line

        opts.desc = "Go to previous diagnostic"
        keymap.set("n", "[d", vim.diagnostic.goto_prev, opts) -- jump to previous diagnostic in buffer

        opts.desc = "Go to next diagnostic"
        keymap.set("n", "]d", vim.diagnostic.goto_next, opts) -- jump to next diagnostic in buffer

        opts.desc = "Show documentation for what is under cursor"
        keymap.set("n", "K", vim.lsp.buf.hover, opts) -- show documentation for what is under cursor

        opts.desc = "Restart LSP"
        keymap.set("n", "<leader>rs", ":LspRestart<CR>", opts) -- mapping to restart lsp if necessary
      end,
    })

    -- used to enable autocompletion (assign to every lsp server config)
    local capabilities = cmp_nvim_lsp.default_capabilities()

    -- Change the Diagnostic symbols in the sign column (gutter)
    -- (not in youtube nvim video)
    local signs = { Error = " ", Warn = " ", Hint = "󰠠 ", Info = " " }
    for type, icon in pairs(signs) do
      local hl = "DiagnosticSign" .. type
      vim.fn.sign_define(hl, { text = icon, texthl = hl, numhl = "" })
    end

    mason_lspconfig.setup_handlers({
      -- default handler for installed servers
      function(server_name)
        lspconfig[server_name].setup({
          capabilities = capabilities,
        })
      end,
      ["svelte"] = function()
        -- configure svelte server
        lspconfig["svelte"].setup({
          capabilities = capabilities,
          on_attach = function(client, bufnr)
            vim.api.nvim_create_autocmd("BufWritePost", {
              pattern = { "*.js", "*.ts" },
              callback = function(ctx)
                -- Here use ctx.match instead of ctx.file
                client.notify("$/onDidChangeTsOrJsFile", { uri = ctx.match })
              end,
            })
          end,
        })
      end,
      ["graphql"] = function()
        -- configure graphql language server
        lspconfig["graphql"].setup({
          capabilities = capabilities,
          filetypes = { "graphql", "gql", "svelte", "typescriptreact", "javascriptreact" },
        })
      end,
      ["emmet_ls"] = function()
        -- configure emmet language server
        lspconfig["emmet_ls"].setup({
          capabilities = capabilities,
          filetypes = { "html", "typescriptreact", "javascriptreact", "css", "sass", "scss", "less", "svelte" },
        })
      end,
      ["lua_ls"] = function()
        -- configure lua server (with special settings)
        lspconfig["lua_ls"].setup({
          capabilities = capabilities,
          settings = {
            Lua = {
              -- make the language server recognize "vim" global
              diagnostics = {
                globals = { "vim" },
              },
              completion = {
                callSnippet = "Replace",
              },
            },
          },
        })
      end,
    })
  end,
}
```

*In the code under `mason_lspconfig.setup_handlers` I setup a default for my language servers and some custom configurations for `svelte`, `graphql`, `emmet_ls`, and `lua_ls`. This can vary depending on the languages that you’re gonna be using.*

Navigate to `nvim-cmp.lua` and make the following change to add the lsp as a completion source:

```
return {
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    "hrsh7th/cmp-buffer", -- source for text in buffer
    "hrsh7th/cmp-path", -- source for file system paths
    {
      "L3MON4D3/LuaSnip",
      -- follow latest release.
      version = "v2.*", -- Replace <CurrentMajor> by the latest released major (first number of latest release)
      -- install jsregexp (optional!).
      build = "make install_jsregexp",
    },
    "saadparwaiz1/cmp_luasnip", -- for autocompletion
    "rafamadriz/friendly-snippets", -- useful snippets
    "onsails/lspkind.nvim", -- vs-code like pictograms
  },
  config = function()
    local cmp = require("cmp")

    local luasnip = require("luasnip")

    local lspkind = require("lspkind")

    -- loads vscode style snippets from installed plugins (e.g. friendly-snippets)
    require("luasnip.loaders.from_vscode").lazy_load()

    cmp.setup({
      completion = {
        completeopt = "menu,menuone,preview,noselect",
      },
      snippet = { -- configure how nvim-cmp interacts with snippet engine
        expand = function(args)
          luasnip.lsp_expand(args.body)
        end,
      },
      mapping = cmp.mapping.preset.insert({
        ["<C-k>"] = cmp.mapping.select_prev_item(), -- previous suggestion
        ["<C-j>"] = cmp.mapping.select_next_item(), -- next suggestion
        ["<C-b>"] = cmp.mapping.scroll_docs(-4),
        ["<C-f>"] = cmp.mapping.scroll_docs(4),
        ["<C-Space>"] = cmp.mapping.complete(), -- show completion suggestions
        ["<C-e>"] = cmp.mapping.abort(), -- close completion window
        ["<CR>"] = cmp.mapping.confirm({ select = false }),
      }),
      -- sources for autocompletion
      sources = cmp.config.sources({
        { name = "nvim_lsp"},
        { name = "luasnip" }, -- snippets
        { name = "buffer" }, -- text within current buffer
        { name = "path" }, -- file system paths
      }),

      -- configure lspkind for vs-code like pictograms in completion menu
      formatting = {
        format = lspkind.cmp_format({
          maxwidth = 50,
          ellipsis_char = "...",
        }),
      },
    })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup trouble.nvim

This is another plugin that adds some nice functionality for interacting with the lsp and some other things like todo comments.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `trouble.lua`

Add the following code:

```
return {
  "folke/trouble.nvim",
  dependencies = { "nvim-tree/nvim-web-devicons", "folke/todo-comments.nvim" },
  opts = {
    focus = true,
  },
  cmd = "Trouble",
  keys = {
    { "<leader>xw", "<cmd>Trouble diagnostics toggle<CR>", desc = "Open trouble workspace diagnostics" },
    { "<leader>xd", "<cmd>Trouble diagnostics toggle filter.buf=0<CR>", desc = "Open trouble document diagnostics" },
    { "<leader>xq", "<cmd>Trouble quickfix toggle<CR>", desc = "Open trouble quickfix list" },
    { "<leader>xl", "<cmd>Trouble loclist toggle<CR>", desc = "Open trouble location list" },
    { "<leader>xt", "<cmd>Trouble todo toggle<CR>", desc = "Open todos in trouble" },
  },
}
```

**The code above has been refactored to work with trouble version 3. This is different from the code in the video**

Exit with `:q` and reenter Neovim with `nvim`

## Setup formatting

We’re gonna use `conform.nvim` to setup formatting in Neovim.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `formatting.lua`

Add the following code:

```
return {
  "stevearc/conform.nvim",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    local conform = require("conform")

    conform.setup({
      formatters_by_ft = {
        javascript = { "prettier" },
        typescript = { "prettier" },
        javascriptreact = { "prettier" },
        typescriptreact = { "prettier" },
        svelte = { "prettier" },
        css = { "prettier" },
        html = { "prettier" },
        json = { "prettier" },
        yaml = { "prettier" },
        markdown = { "prettier" },
        graphql = { "prettier" },
        liquid = { "prettier" },
        lua = { "stylua" },
        python = { "isort", "black" },
      },
      format_on_save = {
        lsp_fallback = true,
        async = false,
        timeout_ms = 1000,
      },
    })

    vim.keymap.set({ "n", "v" }, "<leader>mp", function()
      conform.format({
        lsp_fallback = true,
        async = false,
        timeout_ms = 1000,
      })
    end, { desc = "Format file or range (in visual mode)" })
  end,
}
```

Navigate to `mason.lua` and add the following to auto install formatters:

```
return {
  "williamboman/mason.nvim",
  dependencies = {
    "williamboman/mason-lspconfig.nvim",
    "WhoIsSethDaniel/mason-tool-installer.nvim",
  },
  config = function()
    -- import mason
    local mason = require("mason")

    -- import mason-lspconfig
    local mason_lspconfig = require("mason-lspconfig")

    local mason_tool_installer = require("mason-tool-installer")

    -- enable mason and configure icons
    mason.setup({
      ui = {
        icons = {
          package_installed = "✓",
          package_pending = "➜",
          package_uninstalled = "✗",
        },
      },
    })

    mason_lspconfig.setup({
      -- list of servers for mason to install
      ensure_installed = {
        "tsserver",
        "html",
        "cssls",
        "tailwindcss",
        "svelte",
        "lua_ls",
        "graphql",
        "emmet_ls",
        "prismals",
        "pyright",
      },
    })

    mason_tool_installer.setup({
      ensure_installed = {
        "prettier", -- prettier formatter
        "stylua", -- lua formatter
        "isort", -- python formatter
        "black", -- python formatter
      },
    })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup linting

We’re gonna be using nvim-lint to setup linting in Neovim.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `linting.lua`

Add the following code:

```
return {
  "mfussenegger/nvim-lint",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    local lint = require("lint")

    lint.linters_by_ft = {
      javascript = { "eslint_d" },
      typescript = { "eslint_d" },
      javascriptreact = { "eslint_d" },
      typescriptreact = { "eslint_d" },
      svelte = { "eslint_d" },
      python = { "pylint" },
    }

    local lint_augroup = vim.api.nvim_create_augroup("lint", { clear = true })

    vim.api.nvim_create_autocmd({ "BufEnter", "BufWritePost", "InsertLeave" }, {
      group = lint_augroup,
      callback = function()
        lint.try_lint()
      end,
    })

    vim.keymap.set("n", "<leader>l", function()
      lint.try_lint()
    end, { desc = "Trigger linting for current file" })
  end,
}
```

Navigate to `mason.lua` and add the following to auto install linters:

```
return {
  "williamboman/mason.nvim",
  dependencies = {
    "williamboman/mason-lspconfig.nvim",
    "WhoIsSethDaniel/mason-tool-installer.nvim",
  },
  config = function()
    -- import mason
    local mason = require("mason")

    -- import mason-lspconfig
    local mason_lspconfig = require("mason-lspconfig")

    local mason_tool_installer = require("mason-tool-installer")

    -- enable mason and configure icons
    mason.setup({
      ui = {
        icons = {
          package_installed = "✓",
          package_pending = "➜",
          package_uninstalled = "✗",
        },
      },
    })

    mason_lspconfig.setup({
      -- list of servers for mason to install
      ensure_installed = {
        "tsserver",
        "html",
        "cssls",
        "tailwindcss",
        "svelte",
        "lua_ls",
        "graphql",
        "emmet_ls",
        "prismals",
        "pyright",
      },
    })

    mason_tool_installer.setup({
      ensure_installed = {
        "prettier", -- prettier formatter
        "stylua", -- lua formatter
        "isort", -- python formatter
        "black", -- python formatter
        "pylint", -- python linter
        "eslint_d", -- js linter
      },
    })
  end,
}
```

Exit with `:q` and reenter Neovim with `nvim`

## Setup git functionality

### Setup gitsigns plugin

Gitsigns is a great plugin for interacting with git hunks in Neovim.

Open the file explorer with `<leader>ee` (in my config the `<leader>` key is `space`).

Under `plugins` add a new file with `a` and call it `gitsigns.lua`

Add the following code:

```
return {
  "lewis6991/gitsigns.nvim",
  event = { "BufReadPre", "BufNewFile" },
  opts = {
    on_attach = function(bufnr)
      local gs = package.loaded.gitsigns

      local function map(mode, l, r, desc)
        vim.keymap.set(mode, l, r, { buffer = bufnr, desc = desc })
      end

      -- Navigation
      map("n", "]h", gs.next_hunk, "Next Hunk")
      map("n", "[h", gs.prev_hunk, "Prev Hunk")

      -- Actions
      map("n", "<leader>hs", gs.stage_hunk, "Stage hunk")
      map("n", "<leader>hr", gs.reset_hunk, "Reset hunk")
      map("v", "<leader>hs", function()
        gs.stage_hunk({ vim.fn.line("."), vim.fn.line("v") })
      end, "Stage hunk")
      map("v", "<leader>hr", function()
        gs.reset_hunk({ vim.fn.line("."), vim.fn.line("v") })
      end, "Reset hunk")

      map("n", "<leader>hS", gs.stage_buffer, "Stage buffer")
      map("n", "<leader>hR", gs.reset_buffer, "Reset buffer")

      map("n", "<leader>hu", gs.undo_stage_hunk, "Undo stage hunk")

      map("n", "<leader>hp", gs.preview_hunk, "Preview hunk")

      map("n", "<leader>hb", function()
        gs.blame_line({ full = true })
      end, "Blame line")
      map("n", "<leader>hB", gs.toggle_current_line_blame, "Toggle line blame")

      map("n", "<leader>hd", gs.diffthis, "Diff this")
      map("n", "<leader>hD", function()
        gs.diffthis("~")
      end, "Diff this ~")

      -- Text object
      map({ "o", "x" }, "ih", ":<C-U>Gitsigns select_hunk<CR>", "Gitsigns select hunk")
    end,
  },
}
```

Exit with `:q`

### Setup lazygit integration

Make sure you have lazygit installed.

Install with homebrew:

```
brew install jesseduffield/lazygit/lazygit
```

Open Neovim with `nvim .`

Under `plugins` add a new file with `a` and call it `lazygit.lua`

Add the following code:

```
return {
  "kdheepak/lazygit.nvim",
  cmd = {
    "LazyGit",
    "LazyGitConfig",
    "LazyGitCurrentFile",
    "LazyGitFilter",
    "LazyGitFilterCurrentFile",
  },
  -- optional for floating window border decoration
  dependencies = {
    "nvim-lua/plenary.nvim",
  },
  -- setting the keybinding for LazyGit with 'keys' is recommended in
  -- order to load the plugin when the command is run for the first time
  keys = {
    { "<leader>lg", "<cmd>LazyGit<cr>", desc = "Open lazy git" },
  },
}
```

Exit with `:q` and reenter Neovim with `nvim`

## YOU’RE DONE! 🚀
