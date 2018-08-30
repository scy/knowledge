# Tech Debt

The thing you amass when doing stuff "quick and dirty".

## Taxonomy

On the Riot Games Engineering Blog there's an excellent, albeit really long, article ([A Taxonomy of Tech Debt](https://engineering.riotgames.com/news/taxonomy-tech-debt) by Bill “LtRandolph” Clark) about three axes of tech debt severity and types of tech debt. 
I'll try to summarize it here.

It introduces three axes to assess the severity of tech debt:

* **Impact:** How much does it get in the way of people, be it customers or devs?
* **Fix cost:** How hard is it to fix, and how risky?
* **Contagion:** If we allow it to continue to exist, how much will it spread?

Of these, I consider _contagion_ to be one most people tend not to think about too much, but it's at least as important as the others.

The article then defines types of debt:

* **Local:** 
  Low contagion. 
  If the impact is higher than the fix cost, it will probably be fixed at some point. 
  Try not to focus on these too much, even if it hurts the perfectionist in you.
* **MacGyver:** 
  I don't quite get why it's named like that, but it describes a "bad", legacy way to do things that should be phased out in favor of a new, "better" way, e.g. an old logger class that's used everywhere and that should gradually be replaced with a new interface. 
  Bill states that they tend to let engineers fix these as they go. 
  If the new way is easier and nicer, it will win over the old one at some point, like a "good" contagion: 
  _"If a time-pressured engineer […] chooses to move towards the [new thing], then you’re well on your way."_
* **Foundational:** 
  A bad design decision in one of the more central parts of the system. 
  High fix cost, usually high contagion (e.g. if new code often has to use this interface), often high impact as well. 
  Hard to get rid of, and pretty risky. 
  Often, due to the high cost, it might be advisable to just leave it like it is. 
  If you do want to solve this though, the best way might be to create an alternative system/approach of doing things, making it (back-and-forth-)compatible with the old one and that way convert it into MacGyver debt, gradually phasing out the old code.
* **Data:** 
  A problem in the code that causes users to intentionally enter "wrong" data in order to work around it, e.g. a parameter called `percentage` that expects a value between `0` and `1` instead of `0` and `100`. 
  Once there's already data with values like `0.9`, it's hard to migrate to the "correct" version (because `0.9` would be a valid value there as well, but with a different meaning). 
  Highly contagious, often high fix cost. 
  Often you can't fix it with find/replace. 
  Since data isn't reviewed as much as code and lots of (maybe only loosely connected) people create it, it's hard to define a moment in time from which on all data will be interpreted in the new way. 
  To fix it, either introduce a "do it right" flag that toggles how the data is handled, default it to "new way" for new data and set it to "old way" in existing data (i.e. `floatPercentage = true`). 
  Or, just bite the bullet and fix it once and for all. 
  Feature toggles in production can make this less terrifying.

Let me close with two paragraphs of the original article's summary:

> When measuring a piece of tech debt, you can use impact (to customers and to developers), fix cost (time and risk), and contagion. 
> I believe most developers regularly consider impact and fix cost, while I’ve rarely encountered discussions of contagion. 
> Contagion can be a developer’s worst enemy as a problem burrows in and becomes harder and harder to dislodge. 
> It is possible, however, to turn contagion into a weapon by making your fix more contagious than the problem.
>
> Working on _League,_ most of the tech debt I’ve seen falls into one of the 4 categories I’ve presented here. 
> Local debt, like a black box of gross. 
> MacGyver debt, where 2 or more systems are duct-taped together with conversion functions. 
> Foundational debt, when the entire structure is built on some unfortunate assumptions. 
> Data debt, when enormous quantities of data are piled on some other type of debt, making it risky and time-consuming to fix.
