# OpenPGP and GnuPG

## My 2020 OpenPGP setup

A lot of this is based on [Eric Severance’s 2015 tutorial](https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/).

Key features:

* Master key (used for creating/modifying/signing subkeys and the keys of other persons) lives on a separate, airgapped machine.
* Encryption, decryption and signing happens on a YubiKey, e.g. the YubiKey 5 NFC.
* The OpenPGP key on the YubiKey can also be used as your “SSH key”, i.e. a client-side certificate to log you in to remote servers.
* All of the keys are Ed25519 ones (requires [YubiKey firmware 5.2.3](https://www.yubico.com/blog/whats-new-in-yubikey-firmware-5-2-3/) or higher), not RSA.
* The encryption subkey is _not_ generated on the YubiKey (but signing and authentication are). This allows us to have a backup, should the YubiKey break down.

### Building an airgapped machine for key signing

For this, I use a Raspberry Pi.
You can either boot it, as usual, from an SD card, or improve your defense against [evil maid attacks](https://en.wikipedia.org/wiki/Evil_maid_attack) by booting from a hardware-encrypted USB thumb drive with integrated PIN keyboard like the Kensington DataTraveler 2000.

The basic idea is, once this device contains your master key, it can’t ever be connected to any network again.
Exchanging key material should then only be done using USB drives or other means.
(Since “rogue USB” attacks against USB stacks exist as well, other ways of exchanging data might be worth exploring, for example RS232 or [QR codes](https://github.com/seiferteric/qrtun).)

Flash a boot medium for the Pi and start it, connected to a network.
Set up locale, timezone, keyboard layout etc. according to your needs.

Then, install all packages you’re going to need.
Remember, once you’ve generated your key, you don’t want to connect the machine to the internet ever again, so make sure you have everything you need.
I used these commands to have some basic utilities as well as a GnuPG version that can deal with YubiKeys:

```sh
sudo apt update
sudo apt upgrade
sudo apt install build-essential git links qrencode rsync scdaemon vim yubikey-manager yubikey-personalization zip
```

Then, configure GnuPG with some sane defaults, e.g. those from [Riseup’s best practices](https://riseup.net/en/security/message-security/openpgp/gpg-best-practices):

```
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
personal-cipher-preferences AES256 AES192 AES CAST5
cert-digest-algo SHA512
```

### Setting up the YubiKey

Run `gpg --edit-card` to modify the YubiKey’s OpenPGP settings.
An interactive prompt will appear.
The first command you enter there should be `admin`; it unlocks all of the commands.

First of all, make sure to switch the YubiKey to use Ed25519 keys:

```
gpg/card> admin
Admin commands are allowed

gpg/card> key-attr
Changing card key attribute for: Signature key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519
   (4) NIST P-384
Your selection? 1
The card will now be re-configured to generate a key of type: ed25519
Note: There is no guarantee that the card supports the requested size.
      If the key generation does not succeed, please check the
      documentation of your card to see what sizes are allowed.
Changing card key attribute for: Encryption key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519
   (4) NIST P-384
Your selection? 1
The card will now be re-configured to generate a key of type: cv25519
Changing card key attribute for: Authentication key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519
   (4) NIST P-384
Your selection? 1
The card will now be re-configured to generate a key of type: ed25519
```

Next, enter some basic data about yourself.
Note that the “language preference” is defined as up to four two-byte lower-case ISO 639-1 language codes.
So, if your most preferred language is German (`de`), followed by English (`en`), you should set your language preference to `deen`.
No, there are no spaces, commas or whatever in between.
Just a string of up to eight characters.

```
gpg/card> name
Cardholder's surname: Weber
Cardholder's given name: Tim

gpg/card> lang
Language preferences: deen

gpg/card> sex
Sex ((M)ale, (F)emale or space): m
```

**Do not enable KDF** unless you know what you’re up against.
KDF hashes your PIN, so that it’s no longer transferred in plain text via USB or NFC when communicating with the YubiKey.
This sounds good in theory, but currently, software support is poor:

* Yubico’s own `ykman` doesn’t support it in the current 3.1.1 version, keeping you from setting touch policies etc using `ykman openpgp set-touch` and causing “invalid admin PIN” errors which could possibly force you to reset the YubiKey. It will probably be included in the next release though, see [#279](https://github.com/Yubico/yubikey-manager/issues/279) and [#325](https://github.com/Yubico/yubikey-manager/pull/325).
* The popular Android OpenPGP application OpenKeychain doesn’t support it either. The corresponding issue [#2368](https://github.com/open-keychain/open-keychain/issues/2368) is open for two years now.

Once the KDF situation improves, let me know, and I’ll update this guide accordingly.

Next, if you think it’s better, configure the YubiKey to ask for the pin for each signature.
**Note:** This _toggles_ the flag.
Make sure that the `Signature PIN:` setting in the output of `list` is set to what you want afterwards.

```
gpg/card> forcesig
```

If you didn’t do it already, now would be a good time to change the PIN (default `123456`) and admin PIN (default `12345678`).
(Or, if you’re about to set a loooong admin PIN, maybe wait until you’ve created the keys below.)
Both are not limited to numbers!
You can usually enter up to 127 bytes of UTF-8.

There’s also the “unblock” feature where you can set a “reset code”.
As far as I understand it, it allows to to set a code that can _only_ be used to reset a blocked PIN.
This could be useful if you want to allow someone to reset the PIN without disclosing the admin PIN.
I chose not to use it.

```
gpg/card> passwd
gpg: OpenPGP card no. D276000124[…] detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
[pin dialog]
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
[pin dialog again]
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q
```

### Creating the keys

First, generate only the primary key.
This one will not be used in your day-to-day interactions with GnuPG, but only for modifying subkeys and signing the keys of other people.
It will exist only on the airgapped Raspberry Pi.

By default, it will have _Sign_ and _Certify_ capabilities, but we’re going to change that to _Certify_ only.
Basically, _Certify_ is about key management (adding/modifying/revoking subkeys, signing the keys of others etc.) and _Sign_ is about signing emails or files.

Also, [you most likely should leave the “comment” field of your user ID empty](https://debian-administration.org/users/dkg/weblog/97).
Remember, if you want other people to sign your key, they will need to verify that you are who (and what!) the key says, including the comment.

```
$ gpg --expert --full-gen-key
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate
Current allowed actions: Sign Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate
Current allowed actions: Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 6m
Key expires at Fri 01 Jan 2021 10:12:34 PM CET
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Tim Weber
Email address: scy@scy.name
Comment:
You selected this USER-ID:
    "Tim Weber <scy@scy.name>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
[passphrase dialog]
gpg: /home/pi/.gnupg/trustdb.gpg: trustdb created
gpg: key 38ACA93052B3EB9A marked as ultimately trusted
gpg: directory '/home/pi/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/pi/.gnupg/openpgp-revocs.d/144FEA6B3FF0F6EFD035963B38ACA93052B3EB9A.rev'
public and secret key created and signed.

pub   ed25519 2020-07-05 [C] [expires: 2021-01-01]
      144FEA6B3FF0F6EFD035963B38ACA93052B3EB9A
uid                      Tim Weber <scy@scy.name>
```

Next, we’ll create an encryption-only subkey.
We will _not_ generate it on the YubiKey, but on the Raspberry Pi.
That way, we can have a backup in case the YubiKey is lost or breaks or something.

**Note:** I’m using `@.` here to specify which key to edit.
This actually means “find a key with an email address that contains `.`”.
You can only use this if this is a fresh installation of GnuPG and your keyring only contains your own key and no others.

```
$ gpg --expert --edit-key @.
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 12
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 6m
Key expires at Fri 01 Jan 2021 10:28:41 PM CET
Is this correct? (y/N) y
Really create? (y/N) y
[passphrase dialog]
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb  cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> save
```

Now is a good time to make a backup of your public and private keys, since transferring the encryption key to the YubiKey will remove it from your on-disk keyring.

```
$ gpg --export --armor > ~/2020-07-06-public.asc
$ gpg --export-secret-keys --armor > ~/2020-07-06-secret.asc
[passphrase dialog]
```

Next, move the encryption key to the YubiKey.

```
$ gpg --edit-key @.
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb  cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> key 1

sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb* cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> keytocard
Please select where to store the key:
   (2) Encryption key
Your selection? 2
[passphrase dialog]
[admin pin dialog]
sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb* cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
[ultimate] (1). Tim Weber <scy@scy.name>
```

Now, we need to create signature and authentication subkeys.

```
gpg> addcardkey
Signature key ....: [none]
Encryption key....: F2F4 492E 6061 5C4B EB63  A447 2782 54E6 B4D6 F0F6
Authentication key: [none]

Please select the type of key to generate:
   (1) Signature key
   (2) Encryption key
   (3) Authentication key
Your selection? 1
[admin pin dialog]
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 6m
Key expires at Fri 01 Jan 2021 11:51:45 PM CET
Is this correct? (y/N) y
Really create? (y/N) y
[passphrase dialog]
sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb* cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
     card-no: 0006 13316619
ssb  ed25519/1E99BDD0B41C156B
     created: 2020-07-05  expires: 2021-01-01  usage: S
     card-no: 0006 13316619
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> addcardkey
Signature key ....: 44A6 CD9F 0427 8CCB 1DB3  467D 1E99 BDD0 B41C 156B
Encryption key....: F2F4 492E 6061 5C4B EB63  A447 2782 54E6 B4D6 F0F6
Authentication key: [none]

Please select the type of key to generate:
   (1) Signature key
   (2) Encryption key
   (3) Authentication key
Your selection? 3
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 6m
Key expires at Fri 01 Jan 2021 11:52:39 PM CET
Is this correct? (y/N) y
Really create? (y/N) y

sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb* cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
     card-no: 0006 13316619
ssb  ed25519/1E99BDD0B41C156B
     created: 2020-07-05  expires: 2021-01-01  usage: S
     card-no: 0006 13316619
ssb  ed25519/A8C037AB8AE97089
     created: 2020-07-05  expires: 2021-01-01  usage: A
     card-no: 0006 13316619
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> save
```

### Publishing the keys

First of all, let’s store the keyid in an environment variable for easy access in the future.

```
$ gpg --list-keys --keyid-format 0xshort
/home/pi/.gnupg/pubring.kbx
---------------------------
pub   ed25519/52B3EB9A 2020-07-05 [C] [expires: 2021-01-01]
      144FEA6B3FF0F6EFD035963B38ACA93052B3EB9A
uid         [ultimate] Tim Weber <scy@scy.name>
sub   cv25519/B4D6F0F6 2020-07-05 [E] [expires: 2021-01-01]
sub   ed25519/B41C156B 2020-07-05 [S] [expires: 2021-01-01]
sub   ed25519/8AE97089 2020-07-05 [A] [expires: 2021-01-01]
$ echo export KEYID=52B3EB9A >> ~/.bashrc  # or whereever you store your environment variables
$ . ~/.bashrc
```

Next, export the authentication key in SSH format.
How to actually _use_ this exported key then for authentication is currently beyond the scope of this tutorial.
Basically, you’d need to add `enable-ssh-support` in `gpg-agent.conf` and then use `gpg-agent` as a replacement for `ssh-agent`.
Google around, there are lots of howtos.

Also note:
There was a time when there was a utility called `gpgkey2ssh`, but nowadays GnuPG has replaced that tool with a command line option.
So, in order to extract the key _and_ replace GnuPG’s default comment that specifies the serial number of your YubiKey with something more human-readable, try this:

```
$ gpg --export-ssh-key "$KEYID" | awk '{ print $1, $2, "user@yubikey" }' > ~/$KEYID.pub
```

Of course feel free to replace `user@yubikey` with whatever you like, it’s just the SSH key’s comment field.

If you’d like to publish your GPG key on your own webserver, you should write that URL into the key itself:

```
$ gpg --edit-key $KEYID
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb  cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
     card-no: 0006 13316619
ssb  ed25519/1E99BDD0B41C156B
     created: 2020-07-05  expires: 2021-01-01  usage: S
     card-no: 0006 13316619
ssb  ed25519/A8C037AB8AE97089
     created: 2020-07-05  expires: 2021-01-01  usage: A
     card-no: 0006 13316619
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> keyserver
Enter your preferred keyserver URL: http://scy.name/keys/52B3EB9A.asc
[passphrase dialog]
sec  ed25519/38ACA93052B3EB9A
     created: 2020-07-05  expires: 2021-01-01  usage: C
     trust: ultimate      validity: ultimate
ssb  cv25519/278254E6B4D6F0F6
     created: 2020-07-05  expires: 2021-01-01  usage: E
     card-no: 0006 13316619
ssb  ed25519/1E99BDD0B41C156B
     created: 2020-07-05  expires: 2021-01-01  usage: S
     card-no: 0006 13316619
ssb  ed25519/A8C037AB8AE97089
     created: 2020-07-05  expires: 2021-01-01  usage: A
     card-no: 0006 13316619
[ultimate] (1). Tim Weber <scy@scy.name>

gpg> save
```

Then, export the public key and all of its signatures:

```
$ gpg --armor --export "$KEYID" > ~/$KEYID.asc
```

And this is where the tutorial ends, at least for now.

## Adding a size-optimized photo to your public key

I have just finished writing the tutorial above, which is why, for now, I’ll just link to [Creating a small JPEG photo for your OpenPGP key](https://blog.josefsson.org/2014/06/19/creating-a-small-jpeg-photo-for-your-openpgp-key/) by Simon Josefsson.
There are some valuable pointers in it.
