# Python

_Another nice language._

## Embed e-mail addresses directly in source code

If, for whatever reason, you’d like to write e-mail addresses directly into the code (instead of using the boring method of writing them into a string literal), check out the proof of concept called [mailweird](https://gist.github.com/L3viathan/92addec9501969ae628c90b9100f3177) by [L3viathan](https://github.com/L3viathan).

I didn’t have time to fully understand it, but apparently what it’s doing is using a decorator (`@mail`) to replace function’s code by accessing its bytecode using `fn.__code__` and then iterating over the bytecode primitives, doing some stack magic to replace them.
