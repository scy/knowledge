# Vim

When reading this, you might be interested in [my Vim config](https://github.com/scy/dotfiles/tree/master/.vim), too.
I also have a [NeoVim config](https://github.com/scy/dotfiles/tree/master/.config/nvim), but it's mainly just loading the Vim one.

## Enable folding for Markdown files

There are some plugins to enable Vim's folding inside Markdown files.
However, in a sufficiently recent Vim, this functionality is already included, it's just not enabled by default.
Try this:

```viml
let g:markdown_folding=1
```
