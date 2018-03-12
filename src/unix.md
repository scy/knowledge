# Unix

A good OS, an even better philosophy.

## Is `/dev/urandom` Good Enough?

tl;dr: Yes.

There's a lot of debate about whether `/dev/urandom` can be used for things that should be "secure". 
After all, it's just pseudo-random data, right? 
For crypto keys, secrets, unique IDs etc. you should be using `/dev/random` instead, shouldn't you? 
Because that's where the "real" random data is?

No. 
As Thomas Hühn explains in his long but interesting article [Myths about /dev/urandom](https://www.2uo.de/myths-about-urandom/) (found it in this [detailed Information Security Stack Exchange answer](https://security.stackexchange.com/a/3939)), both are implemented as a cryptographically secure pseudorandom number generator (CSPRNG). 
Both are seeded from the system's entropy pool, and (and this is important) **reseeded** over time when new entropy becomes available. 
The only difference is that `/dev/random` blocks when there is not enough entropy available for (re)seeding.

`/dev/urandom` is _not_ simply seeded with the date and MAC address or whatever, it's seeded using the same entropy that `random` is. 
This means that as soon as there has been enough entropy available during the uptime of the system _once_ since booting, it's seeded using good random data. 
(And since Linux stores leftover entropy when shutting down and reuses it when booting, the chance of not having enough randomness is even lower.) 
And I repeat: 
It will even be reseeded once more entropy comes in.

So, `urandom` is only not good enough if your system didn't have enough entropy since booting up. 
How do you know whether it has? 
Well, that's the harder problem. 
Luckily, there's the [`getrandom()` syscall](http://man7.org/linux/man-pages/man2/getrandom.2.html) available in Linux ≥3.17 (with glibc ≥2.25). 
It will use `/dev/urandom` to get random bytes, but block if it hasn't been initialized with enough entropy yet.

Oh, and if you're confused by the man pages: 
That's totally understandable, and there's been a lot of debate about these as well. 
For example, `urandom(4)` once said this:

> A read from the `/dev/urandom` device will not block waiting for more entropy. 
> As a result, if there is not sufficient entropy in the entropy pool, the returned values are theoretically vulnerable to a cryptographic attack on the algorithms used by the driver. 
> Knowledge of how to do this is not available in the current unclassified literature, but it is theoretically possible that such an attack may exist. 
> If this is a concern in your application, use `/dev/random` instead.

Thomas is talking about that in his article as well, and even mentions [this message by the respected Dan Bernstein](https://www.mail-archive.com/cryptography@randombit.net/msg04763.html), in which he basically says that the man page is full of shit. 
Indeed, [in recent versions](http://man7.org/linux/man-pages/man4/random.4.html) it tells a totally different story. 
Quoting:

> The `/dev/random` device is a legacy interface which dates back to a time where the cryptographic primitives used in the implementation of `/dev/urandom` were not widely trusted.

and

> The `/dev/random` interface is considered a legacy interface, and `/dev/urandom` is preferred and sufficient in all use cases, with the exception of applications which require randomness during early boot time; for these applications, `getrandom(2)` must be used instead, because it will block until the entropy pool is initialized.
>
> If a seed file is saved across reboots as recommended below (all major Linux distributions have done this since 2000 at least), the output is cryptographically secure against attackers without local root access as soon as it is reloaded in the boot sequence, and perfectly adequate for network encryption session keys.

Convinced? 
You should be.
