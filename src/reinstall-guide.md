# Reinstall Guide

This document is supposed to list all the stuff you want to think of when reinstalling a machine (or smartphone).
It's basically not that different from “what to back up”, but less focused on long-term, regular processes in favor of a short-term perspective.

* All **Git repositories** should be clean and pushed. On Windows machines, check repos on the host as well as in WSL.
* Are there any open **browser tabs** you don't want to lose? (In _any_ browser?) Consider screenshotting pinned tabs to be able to restore them in the same order.
* Make sure that this isn't the last machine that allows access to your **password manager**.
* Also, since in **Keybase**, devices invite each other, don't kill the last available device, or at least have a paper key available.
* Check some local directories for stuff you want to keep:
  * **Desktop** (if applicable)
  * **Downloads**
  * **Documents**
  * **Music** (unlikely)
  * **Pictures**
  * **Videos** (again, unlikely)
* On a Windows machine, are there any **PuTTY private keys**?
* Do you have the **password to the backup**?
* Write down or securely store **passwords to auto-unlocking external disks**.

Sometimes my machines are backed up automatically, but most aren't.
That's because I keep most of my work

* in (public or private) Git repos
* on an external SSD that's backed up manually or
* synced to multiple devices, which means there's a copy even if one devices dies.

Automatic cloud backups are not the best solution when you're [living in a van](https://github.com/scy/jessie) with a (generous but limited) mobile data plan.

Also, some things are really host-specific and should probably not be backed up (but maybe also not migrated when reinstalling?); others need to be backed up manually anyway, e.g. the keys/passwords to the backup.
These include:

* GPG keychain
* SSH private keys
* SSH `known_hosts` for hosts that I don't keep in my public dotfiles repo
* Borg config, key, password and metadata
* synced directories when they have not been synced (e.g. because you're reinstalling several devices in short succession and don't want to pair a new device to an old one that's going away tomorrow anyway)

This is a command line that can be used as a template for a manual backup, but needs to be adapted and refined to the machine you're running it on.

```sh
cd \
&& tar czv \
    b17 my \
    .ssh/id_* .ssh/*_hosts .ssh/config.d/* \
    .autoborg .*.autoborg \
| gpg --symmetric --no-symkey-cache \
> $(date '+%Y-%m-%d-%H%M%S')-$(hostname)-manual-backup.tar.gz
```
