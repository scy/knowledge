# Git

Awesome version control.

## Revert changes only to a single file

As of the time of writing this, Git doesn't have a command that says "revert the changes that commit `C` introduced to file `F`". 
You can revert the changes `C` did for _all_ the file it touched, or you can check out `F` as it was before `C`, but reverse-applying the patch, and only to one (or more) files requires a bit more effort, as [this Stack Overflow question](https://stackoverflow.com/q/23068790) points out.

The easiest command that's provided there is this:

```sh
git show $C -- $F | git apply -R
```

Note that this won't work if `C` is a merge commit.
