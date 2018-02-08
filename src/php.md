# PHP

A language I’m proficient in. 
A language with horrible design errors. 
A language that can be used professionally nevertheless.

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
