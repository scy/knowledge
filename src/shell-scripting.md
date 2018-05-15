# Shell Scripting

Still one of the best ways to get things done.

## Scripts that can run both on Unix and on Windows

Apparently it's actually possible to write scripts that are both valid POSIX shell and Windows batch files. 
The trick is the colon: 
In batch files, a colon at the beginning of the line starts a jump label for `GOTO` commands. 
In shell scripts, it's a command that does nothing (like `pass` in Python). 
And if you do it like this …

```
:; echo This is POSIX.
:; exit
ECHO This is Windows.
```

… you have a script that works in both worlds.

This is possible because `;` is not a valid character in a batch file label, so the rest of the line is simply skipped by Windows. 
In POSIX, it denotes the start of a new command. 
Without that, the `:` command would eat everything after it as its parameters.

Thanks to the [Stack Overflow answer that explains the technique](https://stackoverflow.com/a/17623721/417040)!

## Best practices and FAQs

The [shellharden project](https://github.com/anordal/shellharden) has a document on [safe ways to do things in bash](https://github.com/anordal/shellharden/blob/master/how_to_do_things_safely_in_bash.md). 
In my opinion, it's too bash-centric. 
You should always try to write scripts that conform to POSIX. 
If your shell script is _so_ sophisticated that you need bash features like arrays, you maybe should use a "real" programming language anyway.

The [BashPitfalls](http://mywiki.wooledge.org/BashPitfalls) article on Greg's Wiki contains a _lot_ of detailed knowledge. 
The [BashFAQ](https://mywiki.wooledge.org/BashFAQ) in the same wiki is also very useful.
