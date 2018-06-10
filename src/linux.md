# Linux

Just another Unix. 
Make sure to have a look at the [Unix](unix.md) page as well.

## Keeping a plain-text terminal from blanking

Just a one-liner. 
Can most likely be improved, but kind of works.

```sh
TERM=linux setterm -blank 0 | sudo tee /dev/tty0
```
