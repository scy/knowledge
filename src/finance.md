# Finance

## Setting up AqBanking

[AqBanking](https://www.aquamaniac.de/rdm/) is a set of tools for talking to German banks using the HBCI and FinTS protocols.

⚠ **Please note:**
This tutorial, which is based on [the official PIN/TAN tutorial](https://www.aquamaniac.de/rdm/projects/aqbanking/wiki/SetupPinTan), is incomplete.
I couldn't get TANs to work, therefore my AqBanking adventures are currently on hold.

To begin, create a new user.
You'll need the bank's 8-digit BLZ, which should be the 5th to 12th digit of your IBAN.

`-N` should apparently be the account holder's name, `-u` is your login user name.
The URL to use for `-s` is specific to your bank; ask them if you don't know it.

This is an example for my [GLS Bank](https://www.gls.de/) login, with `-u` obviously censored:

```sh
aqhbci-tool4 adduser -N 'Tim Weber' -b 43060967 -u 123456789 -t pintan -s 'https://hbci-pintan.gad.de/cgi-bin/hbciservlet' --hbciversion=300
```

`aqhbci-tool4 listusers` should return a "unique id" for the user you've just created; make sure to use that in all the places below where I use `-u 1`.

Next, retrieve some basic information about the user:

```sh
aqhbci-tool4 getsysid -u 1
aqhbci-tool4 listitanmodes -u 1
```

Set your TAN method to something sensible and marked as "available" in the `listitanmodes` command's output, e.g.:

```sh
aqhbci-tool4 setitanmode -u 1 -m 6942
```

Retrieve a list of all available bank accounts:

```sh
aqhbci-tool4 getaccounts -u 1
aqhbci-tool4 listaccounts -v
```

If you'd like to store your PIN in an unencrypted text file in order to run some commands noninteractively (with the `-P` parameter to `aqbanking-cli`), use this command to create an empty example file for you, replacing `gls.pin` with a file name of your choice:

```sh
aqhbci-tool4 mkpinlist -o gls.pin
```

You’ll need to edit that file using your favorite text editor and put the PIN inside.

AqBanking uses what they call a "context file" to store results of requests in a machine-readable (to some degree also human-readable) format.
Make sure to provide it when calling `aqbanking-cli request` (by supplying the `-c` parameter), else you'll be left with the contents of the context file dumped to stdout with no easy way to query them.

```sh
aqbanking-cli -P gls.pin request -c gls.ctx --transactions --balance --fromdate=20200301
```

You can now use commands like `aqbanking-cli listbal -c gls.ctx` or `aqbanking-cli listtrans -c gls.ctx`.
