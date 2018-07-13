# PHP

A language I’m proficient in. 
A language with horrible design errors. 
A language that can be used professionally nevertheless.

## Composer

Composer is PHP's most widely used dependency manager. 
[It has some quirks imho](https://twitter.com/scy/status/1014429532860936192), but it's what we have, and if you follow some simple rules and know the answers to the most common problems, it's not so bad anymore.

### How to update a requirement (or: change the required version)

These are basically two different things. 
I assume that you know that `composer.json` contains version _constraints_ for your dependencies, while `composer.lock` contains the specific version that has been selected that fulfills the constraint. 
Now, _updating_ can mean that you want to get a newer version of a package _without changing the constraints_ or that you want to change the constraints (usually to require a more recent version). 
Let's look at these two cases separately.

#### Updating a package (keeping the constraints)

That's the easier case. 
Suppose you have installed `somedev/theirpackage` in version `1.0.0`, e.g. by running `composer require somedev/theirpackage:^1.0.0`. 
The `require` section of your `composer.json` will list something like `"somedev/theirpackage": "^1.0.0"`. 
Once a new version gets released that still satisfies your constraint (in this case basically "everything that starts with `1.`"; please read up on [how to specify versions](https://getcomposer.org/doc/articles/versions.md) if you don't know what `^` does), you can run `composer update somedev/theirpackage` and will get the new version. 
You can even run `composer update` without any parameters and this will update _all_ packages to the most recent version that still matches the constraints.

Sometimes you might shy away from that though, because you're scared of breaking anything. 
This is usually a sign of badly chosen constraints (like `dev-master` or `0.*`) or lack of automated testing in your application. 
You should try to fix that as soon as possible, it'll make your life easier and you'll sleep better.

#### Changing the constraints (usually to a higher version)

Suppose you want to update `somedev/theirpackage` to the newly released `2.0.0` version, which no longer matches your `^1.0.0` constraint. 
While many people resort to changing the value in `composer.json` using their editor and then running `composer update somedev/theirpackage`, in my experience you end up with less confusion if you use Composer's CLI for as many tasks as possible. 
But `composer update somedev/theirpackage:^2.0.0` doesn't work; in fact, the `update` command doesn't understand version constraints at all.

That's because it's designed for the first use case (from the section directly above) _only_. 
Hacking `composer.json` will usually work, but it's easier to use the right command for that. 
And since what you're doing is changing the constraints, `require` is the command to do it.

So, run `composer require somedev/theirpackage:^2.0.0`. 
Chances are good that this will not work (even though it's basically the right command) and provide you with a message like

```
Your requirements could not be resolved to an installable set of packages.
[…]
  Problem 1
    - somedev/theirpackage 2.0.0 requires someoneelse/a-dependency ^4.0.0 -> satisfiable by someoneelse/a-dependency[4.0.0].
    - Can only install one of: someoneelse/a-dependency[3.0.0, 4.0.0].
    - Installation request for someoneelse/a-dependency (locked at 3.0.0) -> satisfiable by someoneelse/a-dependency[3.0.0].
```

It's definitely not helping that Composer will list the items under "Problem 1" in a seemingly random order. 
Try to read the lines as separate findings or statements and don't pay attention to the order in which they appear, and you might recognize the problem here:

The new `2.0.0` version of `theirpackage` no longer depends on `a-dependency` in version `^3.0.0`, but now `^4.0.0`. 
However, since your `composer.lock` not only contains the packages and versions your application depends on, but also those _its dependencies_ depend on, it will also include an entry for `someoneelse/a-dependency` and specifies that this package is (and has to be) installed at version `3.0.0`. 
Since the `require` command you just used does not allow any already installed package to change, Composer can't update `a-dependency` to another version. 
You simply didn't allow it.

Luckily, there's a flag for that: `--update-with-dependencies`. 
This will allow Composer to update packages that you haven't explicitly named in your command, as long as they are not also a _direct_ dependency of your application (what Composer calls a "root dependency"), i.e. not listed in `require` in your `composer.json`. 
(If you need to update one of these in the process as well, there's the `--update-with-all-dependencies` flag.)

### When to use `--prefer-dist`

This flag will choose a "dist" version (e.g. a tarball served over HTTP) over a "source" version (e.g. a Git repo cloned via SSH) when installing or updating _dev versions_ of packages. 
Note the emphasis: 
`--prefer-dist` only modifies the behavior if your constraints specify something like `dev-master` or a branch name, while a (tagged) version constraint like `^1.3` will be fetched as dist version by default anyway.

I basically only use this flag for internal packages that are available via a) an internal [Satis](https://github.com/composer/satis) server using IP-based authentication and b) an internal GitLab server that requires an SSH password or key passphrase to use. 
In order to be able to run `composer install` without requiring me to enter a password, I use `--prefer-dist`.

Note that there's also `--prefer-source` that does the opposite, i.e. fetch repositories even for tagged version releases.

### When to use `--ignore-platform-reqs`

This flag makes Composer ignore "platform" requirements, i.e. PHP versions and extensions. 
This means that if your `composer.json` says that your application (or one of its dependencies!) requires PHP 7.1, but your local machine only has 7.0, using this flag you can still install the dependencies without Composer complaining. 
Be aware though that Composer doesn't know whether that results in a usable application, and it doesn't bother.

More often, I use this flag when I'm developing on a Mac and I don't have all of the PHP extensions installed that are required. 
But since these extensions are only used in a small part of the application (and I'm not working on that), I don't care. 
Or, maybe I'm working on that part, but I run the application in a Docker container that _does_ have the extension – but my local machine doesn't (and doesn't need to).

You should try to use this flag as seldomly as possible, as it disables some safety checks. 
You should especially not use it with `require` or `update`, because Composer might then select a version of a package that's incompatible with your target environment.

Also, when you're using this flag in a staging or even production environment, you're almost certainly doing something wrong. 
These environments should really have the PHP version installed that you're developing the app for, and also all required PHP extensions.

### Why does Composer require some package?

When installing or updating, Composer might tell you that it couldn't update some package, or has a version conflict, and you don't even know what this package is or what depends on it. 
For this, there's the `composer why` command that, well, explains to you _why_ a certain package is installed.

### How to re-calculate the `content-hash` in `composer.lock`

You usually need to do this when you've had a merge conflict in `composer.lock` and fixed it. 
Composer will then show a message like this:

```
Warning: The lock file is not up to date with the latest changes in composer.json. You may be getting outdated dependencies. Run update to update them.
```

But what if you don't want to run `composer update`, because you don't want to risk breaking anything in your project? 
You just want it to generate a new `content-hash`.

Some googling may lead you to a command like `composer update nothing`. 
This will, well, update nothing, but recreate the lock file. 
However, [`nothing` is no special word](https://github.com/composer/getcomposer.org/issues/92), you can also use any other package name that does not exist. 
The canonical way to do it though is to run `composer update --lock`, which should be available in every version ≥1.5.0.

### Best practices

* Try to _not_ edit `composer.json` by hand.
  For most of the things you do there, there's also a CLI command, and in my experience using that reduces the chance of Composer getting confused.
* Try _really_ hard to not edit `composer.lock` by hand. 
  The only time you should do it is when you have a merge conflict.
* When you change `composer.json` and/or `composer.lock`, add the command(s) you've used to your commit message. 
  That way, when your changes cause problems, people can see what you did and have a better chance of fixing it.
* Don't commit `composer.phar` to your application's repository, because it's quite large (several MB) and you need to update it regularly. 
  There are better ways to do it, [tiny composer installer](https://github.com/fastbill/tiny-composer-installer) being one of them.

### When you screw up

One final hint: 
When your Composer command fails and you're left with hundreds of changed lines in `composer.lock` and you don't know why and you want to try something different, please **revert** at least `composer.lock` to a **known-good state** and try again from there.

The reason behind this is that if your lock file contains broken stuff already, it can happen that the next commands you run, even if they're now the _right_ commands, do something different or bad because of the messed up `composer.lock`. 
Or, they might simply not care about it being messed up, but not fix it either, so you're still left with that broken file.

Use your version control system, go back to where you started, take a deep breath, focus and try again.

## Getting Good Random Data

I see code that uses `mt_rand()` on an array of characters or something like that way too often. 
But also a hash over `microtime(true) . gethostname()` isn't really random data.

Using `/dev/urandom` is nearly always a good idea, and I can recommend [my summary of why you don't need `/dev/random` in most cases, and when you do](unix.md#is-devurandom-good-enough) to get rid of some common misconceptions. 
However, in PHP there's an even easier way, and that's [the `random_bytes()` function](http://php.net/manual/en/function.random-bytes.php), which I can highly recommend. 
It's available since PHP 7, but if you have some 5.6 code to maintain (or an even older version?) there's a polyfill [paragonie/random_compat](https://github.com/paragonie/random_compat) available all the way down to PHP 5.2.

You can pass its results directly to something like `base64_encode()` or `hash()`, or concatenate it with date and FQDN of your server or whatever to make it even more "unique".

## Xdebug

### For PHP 5.6

Some days ago, [Xdebug 2.6.0](https://derickrethans.nl/xdebug-26.html) was released. 
This is the first version that no longer supports PHP 5.6, causing `pecl install xdebug` to break, e.g. in a `php:5.6-apache` Docker container. 
To fix this, as [suggested in docker-library/php#566](https://github.com/docker-library/php/issues/566#issuecomment-362094015), ask PECL to install the last version that supported PHP 5.6: 

```sh
pecl install xdebug-2.5.5
```

Note that if there’ll ever be a bugfix version `xdebug-2.5.6`, you won’t get it. 
Sadly, as far as I can see there’s no way to tell PECL to install `xdebug-2.5.*` or the like. 
If you know of a way, please contact me.
