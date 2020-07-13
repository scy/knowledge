# Windows Subsystem for Linux (WSL)

_It’s what makes Windows usable._

**Note:**
At the time of writing, I’m still using the first-generation WSL.
WSL2 has been released, and it brings several improvements, but as far as I know some things that worked before have not been implemented yet.
Also, I simply lack the time right now to update.

## Using GnuPG and SSH with a smartcard (e.g. YubiKey)

Setting up a YubiKey to use it under WSL is not a particularly easy task, but it’s manageable.
There are some good articles that I’ve used as a basis:
[Yubikey, gpg, ssh and WSL2](https://blog.nimamoh.net/yubi-key-gpg-wsl2/) (although I don’t see anything specific to version 2 of WSL there) and the even larger one [How to use GPG with YubiKey (bonus: WSL)](https://codingnest.com/how-to-use-gpg-with-yubikey-wsl/) explain the setup hands-on.
I’ll show you [my setup](#my-setup) further down below, but first, let’s talk about the issues.

### What’s the problem?

WSL doesn’t support accessing arbitrary USB devices.
Serial ports (`COMn`) can be used (`/dev/ttySn`), but an OpenPGP smartcard like the YubiKey can only be used from the host operating system, i.e. Windows.

However, [Gpg4win](https://gpg4win.de/) supports `gpg-agent`, which _normally_ allows accessing the smartcard over a socket.
This _can_ be accessed from WSL, but not without a lot of hoops to jump through.

On Windows, `gpg-agent` doesn’t use Unix sockets, because for a long time, they were not available on Windows.
([They are now](https://devblogs.microsoft.com/commandline/af_unix-comes-to-windows/), by the way.)
Instead, it creates a TCP socket on `localhost`.
To secure this socket, it requires you to send a 16-byte randomly-generated “password” before the normal traffic.
This password, along with the (random) port number the socket is using, is written to the file name that usually would be the Unix socket.
Like this:

```
$ hexdump -C /mnt/c/Users/scy/AppData/Roaming/gnupg/S.gpg-agent
00000000  35 33 30 30 35 0a 2c 82  a3 d0 bb 41 7b 44 a9 28  |53005.,....A{D.(|
00000010  a6 27 98 2b 7c e7                                 |.'.+|.|
00000016
```

Of course, when trying to point GPG or SSH from inside WSL to these special files, they expect them to be sockets and it won’t work:

```
$ SSH_AUTH_SOCK=/mnt/c/Users/scy/AppData/Roaming/gnupg/S.gpg-agent.ssh ssh-add -L
Error connecting to agent: Connection refused
```

Even the version of OpenSSH that’s nowadays included in Windows doesn’t know how to deal with these files:

```
> set SSH_AUTH_SOCK=%AppData%\gnupg\S.gpg-agent.ssh
> ssh-add -L
error fetching identities: invalid format
```

#### Using SSH only

If all you care about is using SSH with your YubiKey, there’s [`wsl-ssh-pageant`](https://github.com/benpye/wsl-ssh-pageant).
It uses GnuPG’s Pageant support (that’s PuTTY’s equivalent of `ssh-agent`), i.e. you’ll have to add `enable-putty-support` to `gpg-agent.conf` to provide a socket that can directly be used by the SSH in your WSL.

However, `wsl-ssh-pageant` doesn’t support doing GPG operations.
For that, you’ll need another tool.

#### Using GPG only

If you need to do GPG stuff, you’ll have to use a more universal solution, which is also more messy to set up.
Enter `npiperelay`.
It’s a Go tool to connect to Windows named pipes and provide access to them via stdin/stdout.
In conjunction with `socat`, it can be used to talk to named pipes from WSL.

`npiperelay` has been originally written [by Microsoft’s John Starks](https://github.com/jstarks/npiperelay), but (as of June 2020) hasn’t been updated for two years.
Support for the GPG-style TCP-socket-pointer files only exists in [pull request #2](https://github.com/jstarks/npiperelay/pull/2), which had been ignored by Starks for about a year.
When he came back with change requests, the author of that pull request, NZSmartie, didn’t react anymore, and currently, the pull request is abandoned.
Another user, Lex Robinson, created [pull request #6](https://github.com/jstarks/npiperelay/pull/6) that takes NZSmartie’s code and adds the fixes requested by Starks.
Now again, Starks didn’t react for over a year and to this day.
Yay open source.

This means that if you’d like to use `npiperelay`, you’ll have to download the source (preferably [Lex’s `libassuan` branch](https://github.com/Lexicality/wsl-relay/tree/libassuan), I guess), compile it from source (instructions are provided) and use that.
If you’d like to go down that road, check out the howtos I’ve linked above, or have a look at my setup below.

#### Using both GPG and SSH

Now you might think “aha, so once I have `npiperelay` and `socat` running for OpenPGP stuff, I can `enable-ssh-support` in the host’s `gpg-agent.conf` and use it without needing a _third_ tool (`wsl-ssh-pageant`) accessing the Pageant socket?”

Well, I thought so, too.
But you cannot.
Because even if you do the whole dance with nonced TCP instead of Unix sockets, [`enable-ssh-support` in GnuPG’s Windows port is simply broken and won’t be fixed](https://dev.gnupg.org/T4979).
Or, as lead dev Werner Koch explains:

> […] has never been tested and implemented by me in blind flight mode. I don't think this has any future.

#### Have GnuPG fix the issue

All these workaround are only necessary because GnuPG does that strange TCP-pointer-file stuff on Windows.
It’s no longer necessary since `AF_UNIX` sockets are supported under Windows now.

There’s already [issue T3883](https://dev.gnupg.org/T3883) in the GnuPG project, and Werner Koch seems open to doing something about it.
It has a `gpg23` tag, so I assume they’re planning to implement this in GnuPG 2.3.

### My setup

I’ve written more than enough already.
Let’s keep this short.

First of all, if you don’t have an OpenPGP-enabled YubiKey (or other smartcard) yet that you’d like to use, do check out [my 2020 OpenPGP setup](https://github.com/scy/knowledge/blob/master/src/openpgp.md#my-2020-openpgp-setup).
Next, let’s get to setting up your machine.

#### The Windows side

1. Install a recent [Gpg4win](https://gpg4win.de/).
2. Create the file `%APPDATA%\gnupg\gpg-agent.conf` (where `%APPDATA%` is an environment variable on your machine and should be set to something like `C:\Users\your_user\AppData\Roaming`) and put the line `enable-putty-support` into it.
3. Create a shortcut to `"C:\Program Files (x86)\GnuPG\bin\gpg-connect-agent.exe" /bye` in `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup` so that `gpg-agent` will be started as soon as you log in.
4. Run that shortcut (or log out and in again, but why would you do that).

I think that should be it for the Windows side of things.
(Let me know if I’ve missed something!)

#### The WSL side

This is where it gets funny.

1. Install `git`, `golang`, `socat` and `tmux`.
2. `git clone https://github.com/Lexicality/wsl-relay.git && cd wsl-relay && git checkout libassuan`
3. `go get ./...` to install the required dependencies, then `GOOS=windows go build -o "$HOME/bin/npiperelay.exe" .` (assuming you have a `~/bin` directory). In contrast to what other tutorials say, `npiperelay` doesn’t have to be stored in your Windows filesystem (i.e. under `/mnt/c`); the WSL filesystem is just fine. (Technically, the WSL filesystem lives in your Windows filesystem anyway.)
4. Download the latest [`wsl-ssh-pageant` release](https://github.com/benpye/wsl-ssh-pageant/releases) to `~/bin/wsl-ssh-pageant.exe`.

You have all of the required ingredients now.
Running all three required parts (`npiperelay` and `socat` for GnuPG operations as well as `wsl-ssh-pageant` for SSH), in the background, and only once (not for every shell you spawn) isn’t easy.
Feel free to use [my `wsl-gpg-agent.sh` script](https://github.com/scy/dotfiles/blob/master/bin/wsl-gpg-agent.sh) or build something similar.
It will check whether existing sockets are still alive and launch every part that isn’t running yet.
Basically, that’s

1. Use `socat` to create a Unix socket at `~/.gnupg/S.gpg-agent` that, on connection, spawns `npiperelay.exe` to access `%APPDATA%/gnupg/S.gpg-agent`.
2. Use `wsl-ssh-pageant.exe` to create a normal Unix socket in the Windows filesystem that translates between GnuPG’s Pageant implementation and OpenSSH’s agent protocol.

However, I couldn’t for the life of me write it so that it

* can run from `.bashrc`
* doesn’t freeze your shell on startup and
* doesn’t keep your shell from exiting

and I’ve tried every combination of `&`, `setsid`, `nohup` and `disown` I could think of.
Instead, I went for a background tmux session, because that way, tmux will make sure that it runs only once, and if something goes wrong, you can always attach to the session and check it.
Run it like this, for example, from your `.bashrc`:

```sh
tmux new-session -d -s wsl-gpg-agent wsl-gpg-agent.sh >/dev/null 2>&1
```

(Using a systemd user unit would be an alternative, but since this is WSL1, there’s no systemd.)

Also, while editing the `.bashrc`, don’t forget to tell SSH where to find the socket.
My script puts it in your Windows `gnupg` directory, e.g. `/mnt/c/Users/scy/AppData/Roaming/gnupg/S.wsl-ssh-pageant`, and if you do it like that, you can use a `.bashrc` line like this:

```sh
export SSH_AUTH_SOCK="$(wslpath -a "$(cmd.exe /c echo %APPDATA% 2>/dev/null | tr -d '\r')")/gnupg/S.wsl-ssh-pageant"
```

(I’m using `cmd.exe` to access Windows environment variables, and `wslpath` to translate. The `tr` is required to get rid of the `CR` character `cmd.exe` will insert.)

And that’s basically it!
Kill any `gpg-agent` instances that might be running automatically inside your WSL, then reload your `.bashrc`.
`gpg --card-status` should now show your YubiKey (assuming you have it inserted), `ssh-add -L` should list your authentication key (assuming you have that set up), and `gpg --sign` or `--decrypt` or whatever should work.

#### Issues

* [Sometimes, `wsl-ssh-pageant` will stop reacting while eating 100 % CPU.](https://twitter.com/scy/status/1282414463090597889) If this happens to me more often, I’ll debug it.
* The PIN window of the `gpg-agent` running on Windows will appear in the foreground, but not have keyboard focus. This is annoying. If you’ve fixed this, please tell me how.
* Neither `wsl-ssh-pageant` nor `npiperelay` are particularly good at removing their sockets once they die. Which is why my script detects that and tries to do the right thing.
* I’d love to get rid of the `tmux` requirement.

Other than that, I don’t know of any!
It took me _days_ to debug all of this and set it up, but right now, it’s running nicely.
