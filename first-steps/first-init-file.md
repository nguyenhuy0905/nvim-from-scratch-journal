# Neovim setup journey from scratch

- Ah yes, Neovim, the editor where you spend more time configuring than doing
any actual coding.
- These are lazy times, so I decided, why shouldn't I set up Neovim from scratch?

> [!IMPORTANT]
> Disclaimer: I am not a newbie when it comes to configuring Neovim.
> While this is indeed the first
> time setting up Neovim from scratch, I have done a lot of configurations for
> Neovim. So, this is pretty sure not representative of a newbie's experience.

- If in any case, you would want to follow, make sure to have Neovim installed.
I am being lazy here, but a simple Google on "install Neovim on _your OS here_"
would give you a good guide.
  - For the starting setup up until I installed Treesitter and get the Lua
syntax highlighting, I used
a different text editor (Vim) for the files. You can use which ever one you want.
Just not those fancy ones, like MS Word or LibreOffice. Those are more like
processed XML files rather than what we want here.

- With that out of the way, let's begin.

## Remove old artifacts

- During my time using Neovim, I have always used this plugin called "NvChad"
that does a lot of the weightlifting for me. And, yes, that does mean I have
some config files lying around. Now let's wipe them off:

```bash
rm -rf ~/.config/nvim
rm -rf ~/.local/state/nvim
rm -rf ~/.local/share/nvim
# now let's see
nvim
```

- Looking like new now, with absolutely nothing.

## The first file(s)

- First, let's create an entrypoint. Neovim will, by default, look for
~/.config/nvim/init.lua, I think? So, let's get that going.

```bash
# setting up a basic file structure
mkdir -p ~/.config/nvim/ && cd ~/.config/nvim
touch init.lua
```

- So, we have our main entry point. Simple, right.
- I know what's gonna happen next already. But you can play around with it a bit.

```lua
-- inside init.lua
-- set <leader> key to spacebar.
-- so, any key-maps will substitute <leader> for spacebar, if it uses <leader>.
vim.g.mapleader = " "
print("Hello World")
-- now if you save and close this file (:wq), and open nvim again, you're gonna
-- see a nice "Hello world"
```

- There are 2 default paths that Neovim will seek by default. First is the
init.lua file at `~/.config/nvim`. Second is any directories/files inside
`~/.config/nvim/lua`. We will be installing a lot of plugins, so I'd better
keep them organized. Let's go ahead and create some folders to group our stuff.

```bash
mkdir -p ~/.config/nvim/lua && cd ~/.config/nvim/lua
mkdir -p entry plugins core
```

- I created 4 directories here:
  - `~/.config/nvim/lua`, which is where Neovim automatically seek any
  further Lua files
  - Inside this directory, I created 3 other directories:
    - `entry`, which contains another `init.lua` file, for now.
    - `plugins`, which is where all our plugin configurations go.
    - `core`, which contains some basic options like colorscheme or keymapping.

- So, that's the end of the first step. For the next step, I will install the
lazy.nvim plugin manager, then some other plugins.
