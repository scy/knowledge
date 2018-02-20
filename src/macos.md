# macOS

## Salt

I try to manage all of my machines using [https://saltstack.com](Salt). 
On macOS, I find this a bit harder than I expected.

### Setting up as a minion

In my experience, the `pip` installation method as described in [the official installation docs](https://docs.saltstack.com/en/latest/topics/installation/osx.html) doesn't work. 
For one, because on 10.13 you don't have `pip` installed by default, just `easy_install`. 
Trying to use that to install `pip` and/or Salt leads down another rabbit hole. 

Since you'll be needing the development command line utilities (or whatever they're called) anyway, it's not the worst plan to install Homebrew manually first and then use it to install Salt. 
Something like this:

```sh
# Official way to install Homebrew.
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# Then, get Salt.
brew install saltstack
```

However, you then don't have Salt running at the startup automatically. 
Plus, it's not configured. 
I usually don't configure my minions at all, because they should simply connect to a host called `salt` (the default), but on my machine Salt didn't find out its hostname correctly. 
Instead, it used an `.in-addr.arpa` name, which isn't optimal. 
You should therefore first set a minion ID manually:

```sh
sudo mkdir -p /etc/salt && \
echo put-minion-id-here | sudo tee /etc/salt/minion_id
```

Then, in order to start the minion automatically, put [the launchd config they provide](https://github.com/saltstack/salt/blob/develop/pkg/darwin/com.saltstack.salt.minion.plist) to `/Library/LaunchDaemons` and call `launchctl load -w /Library/LaunchDaemons/com.saltstack.salt.minion.plist`.

### Installing software using homebrew

macOS minions will automatically use Homebrew as their `pkg` implementation, so you can simply use `pkg.installed` like you're used to. 
_But not quite._

Homebrew is really flexible when it comes to asking it for a package to install. 
You can use `gpg`, `gnupg` and even `gpg2` for the same package. 
However, `brew list` will return the canonical name, and `pkg.install` will look in that list to find out whether the installation succeeded. 
So, if you don't use the canonical name to install, it'll install the package but will then claim that it wasn't installed and return an error.

To make things worse, [Casks](https://caskroom.github.io) aren't listed in `brew list` at all.

There's a [fix coming up](https://github.com/saltstack/salt/pull/45309), but unfortunately it's not released yet (as of 2018-02-07). 
I've tried to [patch](https://patch-diff.githubusercontent.com/raw/saltstack/salt/pull/45309.diff) my installation manually with [it](https://github.com/saltstack/salt/pull/45309/files), but the patch won't apply. 
Maybe I'll just wait and live with error messages lying to my face.

Lastly, I don't think there's enough documentation on how to install from a cask. 
This causes [all kinds of confusion](https://github.com/saltstack/salt/issues/26414). 
I've had success with a SLS file like this:

```sls
work stuff:
  pkg.installed:
    - pkgs:
      - caskroom/cask/slack
      - caskroom/cask/zoomus
    - taps: caskroom/cask
```

## Quirks

### DHCP Server Overrides the Local Hostname

![Screenshot demonstrating the problem](_img/macos-dhcp-hostname-override.png)

I didn't find out _why_ this happens; I suppose the DHCP server delivers the hostname of a previous device that used the same IP address. 
Note that this happens even though a host name has been set in the System Preferences (Sharing section).

To fix it, the following worked for me:

```sh
sudo scutil --set HostName whatever
```
