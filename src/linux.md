# Linux

Just another Unix. 
Make sure to have a look at the [Unix](unix.md) page as well.

## Keeping a plain-text terminal from blanking

Just a one-liner. 
Can most likely be improved, but kind of works.

```sh
TERM=linux setterm -blank 0 | sudo tee /dev/tty0
```

## APT: Installing things noninteractively

When using `apt` or `apt-get` to install something and you want to make sure that it doesn’t wait for user interaction (for example because you’re calling it from a `Dockerfile`), using `apt install -y` is not enough.
Packages requesting interactive configuration (for example the configuration of the machine’s time zone) will still wait for your reaction, even if there’s nothing connected to stdin (at least that’s how it looked like to me).

The trick is to also set the environment variable `DEBIAN_FRONTEND=noninteractive`.
So, in a `Dockerfile`, you would write something like:

```dockerfile
FROM ubuntu

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update \
    && apt install -qy whatever
```
