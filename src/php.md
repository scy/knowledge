# PHP

A language I’m proficient in. 
A language with horrible design errors. 
A language that can be used professionally nevertheless.

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
