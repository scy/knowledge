# Documenting Software

## Architectural Decisions

This is basically taken from the good ideas at [Documenting architecture decisions, the Reverb way](https://product.reverb.com/documenting-architecture-decisions-the-reverb-way-a3563bb24bd0).

* Store architectural decisions in your codebase.
* Document _thoughts_ and _decisions_, not the state of things. 
  Even when the state changes (i.e. documentation drift), you are still able to find out what your plan was and (even more important) _why_ that was your plan.
* Include a summary (tldr) section and one that goes into details.
* Include a tags/SEO section, where you put things like class and function names, business concepts etc. 
  This allows you (and others!) to find the documentation by accident when grepping.
* Link to these documents from comments in the code.
