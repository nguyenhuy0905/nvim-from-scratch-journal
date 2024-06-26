# Lazy plugin manager and first plugins

> [!NOTE]
> I wrote this part after forgetting about this, and already finished the setup.
>

## Install lazy.nvim

- Lazy is a package loader. It install packages you declare, then load
them when necessary (aka, lazy-loading), and is my go-to package
loader.
- I added the following to the `init.lua` file at the top:

```lua
-- init.lua
if not (vim.uv or vim.loop).fs_stat(lazypath) then
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

- Then, set it up. In here, I was too hopeful, in wishing that the VSCode
Neovim extension would work, so I separated my entry files into 3 smaller
files.
- I should have sticked with not trying to use NeoVim inside VS Code, and
instead put all my plugins declarations inside one single file. But, here
we go.

```lua
-- entry/init.lua
require("lazy").setup({
    {
        -- non-compatible with Nvim-vscode,
        -- or at least so I thought
        import = "entry.novscode",
        cond = function()
            return not vim.g.vscode
        end,
    },
    {
        -- only compatible with Nvim-vscode,
        -- which, I found none
        import = "entry.vscode",
        cond = function()
            return vim.g.vscode
        end,
    },
    {
        -- always on
        import = "entry.alwayson",
        cond = true,
    },
})
```

```lua
-- init.lua
-- load the init.lua file in entry directory
require("entry")
-- set up mappings
require("core.mappings")
-- set up options
require("core.options")
-- add/modify some commands
require("core.cmd")
```

- Using a centralized "mapping" file probably defeats the purpose of
lazy-loading, since now I may accidentally load a plugin. When I
don't want to, by pressing the wrong key combination. Also, this
doesn't let me assign the same keybinds to plugins that will never
be active at the same time. I am planning to move the plugin-specific
mappings into their separate config files. But, a cental "mapping" file
like this makes it easier to find keybinds, especially since I have
somewhere about 30 individual config files for plugins.

## "Essential" plugins

### Treesitter

- This is the plugin that allows for syntax highlighting. Without
it, one only has, like, gitcommit or yaml being highlighted.
- So, let's add it in:

```lua
-- lua/entry/novscode.lua
return {
...
{
    "nvim-treesitter/nvim-treesitter",
    event = BUFOPEN,
    config = function()
        require("plugins.treesitter")
    end,
},
...
}
```

- Since I specifically said my configs for this plugin is inside the
directory `plugins/treesitter.lua`:

```lua
-- lua/plugins/treesitter.lua
require("nvim-treesitter.configs").setup({
ensure_installed = {
    -- low level
    "c",
    "cpp",
    "rust",
    -- the pain
    "cmake",
    "make",
    -- java and friends
    "c_sharp",
    "java",
    -- obviously,
    "lua",
    "bash",
    "vim",
    "vimdoc",
    -- web and friends
    "html",
    "css",
    "javascript",
    "typescript",
    "json",
    "markdown",
    "markdown_inline",
    "sql",
    "xml",
    -- git
    "git_config",
    "git_rebase",
    "gitattributes",
    "gitcommit",
    "gitignore",
    -- forced to use
    "python",
    -- config files
    "yaml",
    "ini",
},
})
```

- After this, quit and reopen Neovim. You should see the parsers listed getting installed.
And when you open a Lua file (say, init.lua), you can see colorful text now. Yay.

### Mason

- Yet, another package manager, but for a slightly different purpose. While I certainly
can use Lazy to install language-specific packages (eg, LSP, linter, formatter.
Apart from parsers, which is Treesitter's job). Mason makes it much simpler.
If I recall correctly, it not only just clones the package(s) from GitHub, but
also compiles and builds the package into an executable for you.
- I will use Mason to install all the LSP, linter and formatter I need. But first,
let's set Mason up.

```lua
-- lua/entry/alwayson.lua
{
    "williamboman/mason.nvim",
    config = function()
        require("plugins.mason")
    end,
},
```

- I also set up a convenient table and function to install all the packages listed
with only 1 command. This is a trick I learned when I was using a pre-made Neovim
distribution, NvChad.

```lua
-- lua/plugins/mason.lua
MASON_INSTALLS = {
    -- DAP
    "netcoredbg",
    "codelldb",
    -- Linter
    "vale",
    "codespell",
    "markdownlint",
    "cmakelang",
    -- clang-tidy isn't here, but we can use it if we downloaded the Clang toolchain
    -- Formatter
    "stylua",
    "shfmt",
}

require("mason").setup()
```

- You can find package names by using the command `Mason`.
- In case someone doesn't know, you can type a command in normal mode (entred by
pressing Esc), then `:command-name`. In this case; `:Mason`.

```lua
-- lua/core/cmd.lua
local cmd = vim.api.nvim_create_user_command

-- i learned this one from NvChad
cmd("MasonInstallAll", function()
    if #MASON_INSTALLS == 0 then
        print("No packages to install!")
        return
    end
    local mr = require("mason-registry")
    mr.refresh(function()
        for _, inst in ipairs(MASON_INSTALLS) do
            local pkg = mr.get_package(inst)
            if not pkg:is_installed() then
                pkg:install()
                print("Installing " .. inst)
            else
            end
        end
    end)
end,
{ desc = "Install missing packages defined in MASON_INSTALLS table" })
```

- I also set up another Mason package, `mason-lspconfig`. I could have put all
my LSP installs inside the MASON_INSTALLS table, but when I was setting
everything up, I have yet to think of this method.

### LSP

- Prior to setting up Neovim, I never thought much about those red squiggly
warnings that my text editor/IDE generates, and took it for granted. Turns
out, the things in the background enabling such features is called a "language
server protocol", or LSP. A LSP indexes and reads through your code to give you
hints and raise errors (those red squiggly lines), warnings, and code suggestions.
- Setting up LSP in Neovim is surprisingly simple, if your LSP is supported
by this plugin called `nvim-lspconfig`.
