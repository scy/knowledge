# Shell Scripting

Still one of the best ways to get things done.

## In-place processing of files (like `sed -i`)

On GNU systems, `sed` has a `-i` flag that will cause it to read from the given input file and write the result to the same file. 
They call that "in-place editing"; what actually happens is that the output is written to a new file with a generated name and, once completed, that file will be moved over the old one (or, if you use the optional "backup file suffix", the old one will be renamed first).

On macOS, `sed` also has `-i`, but requires passing the backup suffix. 
To be fair, you can pass an empty value using `sed -i ''`, but this style is incompatible with GNU `sed`. 
See [sed in-place flag that works both on Mac (BSD) and Linux](https://stackoverflow.com/q/5694228) on Stack Overflow for details.

This makes it impossible to have a cross-platform `sed` invocation with `-i` and without using a suffix. 
([Using a suffix and deleting the backup file](https://stackoverflow.com/a/22084103) is the obvious workaround.)
However, you can cheat by using a construct like this:

```sh
{ rm file.txt; sed '…' > file.txt ; } < file.txt
```

This works because on Unix, a deleted file will still be available as long as there are still open file handles to it, and the `< file.txt` gets interpreted before `rm file.txt`.

As you can see, we no longer depend on sed's `-i` flag, so you can use this trick with other commands, too!

**Be careful though:** If the command that follows the `rm` fails to run for some reason, your file will be gone; there's no safety.

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

Google has published their [bash style guide](https://google.github.io/styleguide/shell.xml), and yes, it's explicitly about bash. 
They even disallow the use of `/bin/sh`.
