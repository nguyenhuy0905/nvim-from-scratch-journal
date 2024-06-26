# Lazy plugin manager and first plugins

> [!NOTE]
> I wrote this part after forgetting about this, and already finished the setup.
>

## Install lazy.nvim

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

```lua
-- init.lua
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

```
