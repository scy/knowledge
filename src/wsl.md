# Windows Subsystem for Linux (WSL)

_It’s what makes Windows usable._

**Note:**
At the time of writing, I’m still using the first-generation WSL.
WSL2 has been released, and it brings several improvements, but as far as I know some things that worked before have not been implemented yet.
Also, I simply lack the time right now to update.

## Using GnuPG and SSH with a smartcard (e.g. YubiKey)

**Note:**
I’m not going to go into detail about how to set everything up, because others have done a fine job already.
There are [Yubikey, gpg, ssh and WSL2](https://blog.nimamoh.net/yubi-key-gpg-wsl2/) (although I don’t see anything specific to version 2 of WSL there) and the even larger article [How to use GPG with YubiKey (bonus: WSL)](https://codingnest.com/how-to-use-gpg-with-yubikey-wsl/) that explain setup hands-on.
Here, I’ll rather provide an overview of the underlying issues.

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

### Using SSH only

If all you care about is using SSH with your YubiKey, there’s [`wsl-ssh-pageant`](https://github.com/benpye/wsl-ssh-pageant).
It uses GnuPG’s Pageant support (that’s PuTTY’s equivalent of `ssh-agent`), i.e. you’ll have to add `enable-putty-support` to `gpg-agent.conf` to provide a socket that can directly be used by the SSH in your WSL.
It’s pretty easy to set up.

However, `wsl-ssh-pageant` doesn’t support doing GPG operations.
For that, you’ll need another tool.

### Using GPG _and_ SSH

If you need to do GPG stuff too, you’ll have to use a more universal solution, which is also more messy to set up.
Enter `npiperelay`.
It’s a Go tool to connect to Windows named pipes and provide access to them via stdin/stdout.
In conjunction with `socat`, it can be used to talk to named pipes from WSL.

`npiperelay` has been originally written [by Microsoft’s John Starks](https://github.com/jstarks/npiperelay), but (as of June 2020) hasn’t been updated for two years.
Support for the GPG-style TCP-socket-pointer files only exists in [pull request #2](https://github.com/jstarks/npiperelay/pull/2), which had been ignored by Starks for about a year.
When he came back with change requests, the author of that pull request, NZSmartie, didn’t react anymore, and currently, the pull request is abandoned.
Another user, Lex Robinson, created [pull request #6](https://github.com/jstarks/npiperelay/pull/6) that takes NZSmartie’s code and adds the fixes requested by Starks.
Now again, Starks didn’t react for over a year and to this day.
Yay open source.

This means that if you’d like to use `npiperelay`, you’ll have to download the source (preferably [Lex’s branch](https://github.com/Lexicality/wsl-relay/tree/libassuan), I guess), compile it from source (instructions are provided) and use that.
If you’d like to go down that road, check out the howtos I’ve linked above.

Maybe `wsl-ssh-pageant` author Ben Pye might be interested to add the `gpg-agent` feature to his tool, since he seems rather active.
I guess I’ll ask him if I find the time.
