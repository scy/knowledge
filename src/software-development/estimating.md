# Estimating Software Development

## Coarse estimation

I found some interesting ideas in [Gib mir eine Zahl – Schätzungen entlang des Entwicklungsprozesses](https://www.heise.de/developer/artikel/Gib-mir-eine-Zahl-Schaetzungen-entlang-des-Entwicklungsprozesses-4119174.html?seite=all) by [Annegret Junker](https://twitter.com/grinseteddy):

Estimate initial ideas (at the beginning of a project) more roughly, for example with t-shirt sizes (XS, S, M, L, XL). 
You can't really estimate them down to the hour anyway, and you shouldn't.

If you're having trouble assigning a shirt size to a task, simply use "M" for the first one and then place the others relative to it. 
If something would need to go below XS or above XL, consider moving all of the tasks a level up or down.

To get a time estimate from that, take a past reference project or task (not too large, not too small) that you know the time effort for and give it a shirt size. 
Now, all of your shirt sizes can be roughly estimated using that reference time. 
A suggestion for scaling the sizes is by using a Fibonacci-ish factor, with XS being 1, S being 2, M 3, L 5 and XL 8. 
So, if your reference project is placed in "S" and took seven sprints, an XL project would take 8*(7/2)=28 sprints.

You can use a similar estimation technique for functional parts of a project: 
Start with one part and place the others next to it: 
above it if they're larger, below if they're smaller. 
Place other items in between if that's where they belong. 
Same-sized items can be placed next to each other. 
Now, start with a part that you can easily estimate and let the team give it a Fibonacci number using planning poker. 
Continue pokering in the same row; if you find out that some parts are actually more or less complex than initially thought, move them up or down accordingly. 
Continue with the other rows.

Don't consider risk or additional effort (like learning a technology) in complexity numbers; instead, these two estimates should be additional metadata to write on the cards (or in the tickets). 
When prioritizing, these should be considered, but don't skew your estimates by mushing every factor together in one number.
