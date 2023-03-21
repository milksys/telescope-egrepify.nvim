# telescope-egripfy.nvim

In brief, this [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)-extension is my personal alternative to [telescope-live-grep-args.nvim](https://github.com/nvim-telescope/telescope-live-grep-args.nvim).

## Features

- Extensible prefix-based CLI parsing for `ripgrep`, e.g. by default `#md,lua sorter` would look for sorter in files with `md` or `lua` extensions
- Custom entry maker to parse `ripgrep` json to separate filenames as "section titles" (inspired by [consult.el](https://github.com/minad/consult)), configure line and column numbers, and perform accurate line highlighting
- Usable defaults the author likes like `AND` operator for tokens in prompt

## Motivation

Fuzzy-search is often not great at filtering results. 99% of my time, I mostly want to restrict searches by `AND` (intersection of words in search prompt) and files. `ripgrep` syntax is not _as_ trivial as `fzf` but with, e.g., helpers like `telescope-egripfy`'s prefixes enables more precise finding and filtering.

![Screenshot](https://user-images.githubusercontent.com/39233597/226608982-b3400cea-3aca-499c-afb3-51912443a240.png)
The screenshot shows searching for `require` only in files with `md` extension (i.e. markdown files). For more prefixes and configuration, see [Prefixes](#prefixes).

# Prefixes

The core functionality of `telescope-egripfy.nvim` are `prefixes`. The below prefixes are the builtin-defaults. Should you want to use this extension, please __carefully__ read the table below on how a prefix is added.  The configuration section shows how to add another prefix.

```lua
-- DEFAULTS
-- filter for file suffixes
-- example prompt: #lua,md $MY_PROMPT
-- searches with ripgrep prompt $MY_PROMPT in files with extensions lua and md
-- i.e. rg --glob="*.{lua,md}" -- $MY_PROMPT
{ ["#"] = {
      -- #$REMAINDER
      -- # is caught prefix
      -- `input` becomes $REMAINDER
      -- in the above example #lua,md -> input: lua,md
      flag = "glob",
      cb = function(input)
          return string.format([[*.{%s}]], input)
      end,
  },
  -- filter for (partial) folder names
  -- example prompt: >conf $MY_PROMPT
  -- searches with ripgrep prompt $MY_PROMPT in paths that have "conf" in folder
  -- i.e. rg --glob="**/conf*/**" -- $MY_PROMPT
  [">"] = {
      flag = "glob",
      cb = function(input)
          return string.format([[**/{%s}*/**]], input)
      end,
  },
  -- filter for (partial) folder names
  -- example prompt: &egrep $MY_PROMPT
  -- searches with ripgrep prompt $MY_PROMPT in paths that have "egrep" in file name
  -- i.e. rg --glob="*egrep*" -- $MY_PROMPT
  ["&"] = {
      flag = "glob",
      cb = function(input)
          return string.format([[*{%s}*]], input)
      end,
  }
}
```
If you want to opt-out of a prefix you can set `prefix` or pass `prefix` to `opts` with the corresponding prefix character set to `false`:

```lua
-- opting out of file extension
{ ["#"] = false }
```

See also `Configuration`.

# Installation

Here is one way to install this extension with [lazy.nvim](https://github.com/folke/lazy.nvim).

```lua
{
    "fdschmidt93/telescope-egrepify.nvim",
    dependencies = { "nvim-telescope/telescope.nvim", "nvim-lua/plenary.nvim" }
}
```

Make sure to

```lua
local opts = { "$YOUR_TELESCOPE_OPTS" }
require "telescope".setup()
require "telescope".load_extension "egrepify"
```

to appropriately setup the plugin.


## Usage

You can call this extension with vimscript

```vim
:Telescope egrepify
```

or lua

```lua
require "telescope".extensions.egrepify.egrepify {}
```


# Configuration

It will allow you to easily build custom functionality for shorthands to create `ripgrep` flags on-the-fly seamlessly.

```lua
require("telescope").setup {
  extensions = {
    egrepify = {
      -- intersect tokens in prompt ala "str1.*str2" that ONLY matches
      -- if str1 and str2 are consecutively in line with anything in between (wildcard)
      AND = true,                  -- default
      lnum = true,                 -- default, not required
      lnum_hl = "EgrepifyLnum",    -- default, not required, links to `Constant`
      col = false,                 -- default, not required
      col_hl = "EgrepifyCol",      -- default, not required, links to `Constant`
      title_hl = "EgrepifyTitle",  -- default, not required, links to `Title`
      -- suffix = long line, see screenshot
      -- EXAMPLE ON HOW TO ADD PREFIX!
      prefixes = {
        -- ADDED ! to invert matches
        -- example prompt: ! sorter
        -- matches all lines that do not comprise sorter
        -- rg --invert-match -- sorter
        ["!"] = {
          flag = "invert-match",
        },
      },
    },
  },
}
require("telescope").load_extension "egrepify"
```

# DISCLAIMER

Please consider forking for your own customization or well-formed PRs instead to fix issues or add new features. This extension foremost serves my own needs and turned into a plugin as maybe other users may want to personalize `rg` via `telescope.nvim` in similar fashion. While many options _are_ configurable, frankly, I will not promise long-term maintenance beyond review (and hopefully merging) of PRs.

# Naming

Kudos to ChatGPT:

> `egrepify` combines the concept of "grep" (a common Unix command for searching through files) with the word "Epsilon" (the fifth letter of the Greek alphabet, which can represent "my" in mathematical notation). The resulting word suggests a personalized or customized version of grep.
