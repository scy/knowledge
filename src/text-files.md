# Everything(?) as Text Files

I'm a huge fan of using plain text files for things, because they

* can easily be version tracked using Git
* can be edited with builtin (okay, except Android) software on every operating system
* can be modified automatically using a wide variety of tools

This document is supposed to be a collection of things you can do with text files and tools that work with them.

## Diagrams, Graphs etc.

The [Asciidoctor Diagram](https://asciidoctor.org/docs/asciidoctor-diagram/) page serves quite well as a list of tools to create diagrams ([thanks Martin](https://twitter.com/bountin/status/1184110133749661696)).
Here are those that I found most interesting:

* Tools that create diagrams based on a textual description:
  * [PlantUML](http://plantuml.com/) is nice, even if the “UML” part scares you (SVG, PNG, LaTeX output)
    * there's an interesting [article about integration with Markdown, VS Code and GitHub](https://blog.anoff.io/2018-07-31-diagrams-with-plantuml/), [thanks hadez](https://chaos.social/@hadez/102968490447571876)
  * [TikZ](http://www.texample.net/tikz/) is a LaTeX package and very advanced and versatile, but also complex
  * [GraphViz](http://www.graphviz.org/), but its [DOT](https://en.wikipedia.org/wiki/DOT_(graph_description_language)) language is somewhat limited
  * [mermaid](https://mermaidjs.github.io/) can render flowcharts, sequence, Gantt and class diagrams in a browser; [its CLI component](https://github.com/mermaidjs/mermaid.cli) relies on a headless Chrome
* Tools that create diagrams based on ASCII art:
  * [Shaape](https://github.com/christiangoltz/shaape)
  * [ditaa](http://ditaa.sourceforge.net/)
