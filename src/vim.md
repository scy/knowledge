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

## VimWiki

[VimWiki](https://vimwiki.github.io/) is a nice plugin to create a personal wiki inside of Vim.
Some helpful links:

* [VimWiki cheatsheet](http://thedarnedestthing.com/vimwiki%20cheatsheet)
* [patrickdavey's config](https://github.com/patrickdavey/dotfiles/blob/682e72e4b7a70e50858d1a3b7f0713ce6b470fb6/vim/.vimrc#L243-L281) demonstrating multiple wikis, nested syntaxes, and using Markdown with folding
* a [Hacker News thread about VimWiki](https://news.ycombinator.com/item?id=13157497) with some additional hints
