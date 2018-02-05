# Chrome OS

Hints for working with Google’s high-security, browser-only operating system.

## SSH authentication via SmartCards (YubiKey, Nitrokey etc.)

I was surprised that this is possible at all, let alone _this_ easy.

Note that I assume that you already have a key on your SmartCard that can be used for SSH authentication, and that the remote machine has this key in its `authorized_keys` file.

1. Get the [Secure Shell](https://chrome.google.com/webstore/detail/secure-shell/pnhechapfaindjhompbnflcldabbghjo) and [Smart Card Connector](https://chrome.google.com/webstore/detail/smart-card-connector/khpfeaanjngmcnplbdlpegiifgpfgdco) applications. 
   Both are from Google, so you don’t need to trust third party devs. 
   Thanks, Google!
2. In the connection settings of the Secure Shell app, add `--ssh-agent=gsc` (for Google Smart Card, I guess?) to the SSH relay server options (not the OpenSSH options).
3. Connect. 
   (When you use the Smart Card Connector for the first time, you’ll be asked whether Secure Shell should be allowed to access it.) 
   You’ll be asked for the SmartCard’s PIN in the terminal while the connection is established.

Seriously, this is way easier than the stuff you have to do with `gpg-agent` on Linux machines. 
And have you tried getting it to work on Windows? 
Ha. Good luck with that …

Oh, and Google even made a [simple, concise document explaining how to do it](https://chromium.googlesource.com/apps/libapps/+/master/nassh/doc/hardware-keys.md) that I’ve basically just could have linked to.
