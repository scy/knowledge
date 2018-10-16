# International PHP Conference 2018

October 15 to 19, Munich; [phpconference.com](https://phpconference.com/)

These are some notes I took during the talks. They don't necessarily contain everything the presentation was about, but what I personally found noteworthy. I didn't check facts and figures for accuracy. I might provide sources to some statistics or whatever at a later point.

## [Is a new Cross-platform Development Era coming?](https://phpconference.com/web-development/is-a-new-cross-platform-development-era-coming/)

by Maxim Salnikov ([@webmaxru](https://twitter.com/webmaxru))

* PWAs use modern web APIs along with traditional progressive enhancement to create cross-platform applications.
* more mobile usage than desktop
* more than 50% of the users install **zero** apps per month
* web technologies on mobile are improving: performance, access to hardware, auth & payments, integration with the OS: [whatwebcando.today](https://whatwebcando.today/)
* offline PWAs work since Service Worker API
* Why should we call a PWA an "app" and not a website? There are 3x more mobile web unique visitors compared to app users, but people spend 20x more time in apps than on the web
* check out: [Workbox](https://workboxjs.org/), [PWA Builder](https://www.pwabuilder.com/)
* problems:
  * breaking changes in the APIs do happen
  * browser support is still spotty
  * there's no shared roadmap between browser vendors
* success stories (see [PWA Stats](https://www.pwastats.com/))
  * Twitter Lite: 3% size of Android version, 70% data usage reduction
  * mobile Uber: 50k gzipped, takes less than 3s on 2G networks
  * Lancome: 17% conversion increase
* Gartner predicts that PWAs will have replaced 50% of general-purpose, consumer-facing apps by 2020

## [Do you verify your Views?](https://phpconference.com/testing-quality/do-you-verify-your-views/)

by Arne Blankerts and Sebastian Bergmann

* a view could be a HTML page, XML doc, JSON blob; the markup; a fragment; the data and content shown – or the code that generates them
* a view is **code**
* if it's code, we can test it, and if we can test it, we _should_ test it
* could be done manually by looking at pages (technically an end-to-end test)
* automated with Selenium (still end-to-end)
* end-to-end tests relies on a lot of things in addition to the actual view: webserver, database, routing, template engine, …
* they are also slow and have a high maintenance cost, and it's harder to find the cause of an error if a test fails (compared to a unit test for example)
* edge-to-edge testing would leave out the webserver and browser, i.e. fake a request programmatically
  * works even with legacy code by setting `$_REQUEST` etc.
  * there are XPath extensions for PHPUnit, see [Domain-Specific Assertions presentation](https://thephp.cc/dates/2017/10/symfony-live-berlin/domain-specific-assertions)
* invoking the action directly, you got rid of (most of the) framework code and routing
* next, we could mock the database
  * there's DBUnit if you're using for PHPUnit, but it's unmaintained and its future not too bright, all large frameworks don't use it
  * Sebastian tends to suggest using your own custom solution
* skip the action; you now provide sample data and pass it to the template engine that provides it to the view
* skipping the template engine as well usually isn't going to work
  * templating today usually has a "push architecture", where the view receives basically an array of crap`^W`data
  * templates are code: formatting logic, decision logic, iterations, security (escaping)
  * you have no idea what data that you pass in will actually be _used_ and which will be considered for decisions
    * side note: do you filter information that you pass into a template (like user data that should only be admin-visible)?
* there's [this blog post from 2011](http://www.workingsoftware.com.au/page/Your_templating_engine_sucks_and_everything_you_have_ever_written_is_spaghetti_code_yes_you) showing that this problem isn't new …
* calling additional business logic from _inside_ the template makes it even worse
* there's a presentation by Nikolas Martens: [Templating – You're doing it wrong](https://www.youtube.com/watch?v=bi0Cb97f7G4)
* it sparked [Tempan](https://github.com/watoki/tempan), which uses HTML annotated using [RDFa](https://en.wikipedia.org/wiki/RDFa)
  * the view "model" then contains methods for every `attribute` in the HTML template that returns whatever data the template needs
  * these methods can also return objects like users, links etc.
  * the view models are now unit-testable!
  * pull architecture, no template engine needed, no template required, no framework needed
  * you can just instantiate the view
* CQRS is basically going in the same direction
* Arne created [Templado](https://templado.io/) that uses that "view model" concept
  * it also has an in-development PHPUnit integration that traces which methods are being called, to make sure you don't have unconsumed data in your view model
  * it also has JSON view support upcoming
* **to summarize:** separate HTML/JSON and template logic; use "view models" that can be tested without templating engines and can be traced

> If you cannot unit-test it, you cannot reuse it.
> — Sebastian Bergmann

## [Ten standards a PHP developer should know](https://phpconference.com/php-development/10-standards-a-php-developer-should-know/)

by Sebastian Feldmann

* [PHP Framework Interop Group](https://www.php-fig.org/)
* PSR vs [RFC](https://wiki.php.net/rfc/howto)
* PSR-1/PSR-2/PSR-12: coding standards
* PSR-0/PSR-4: autoloading
  * [`__autoload`](http://php.net/manual/en/function.autoload.php) is deprecated since 7.2
* PSR-3: logging
* PSR-6/PSR-16: caching
  * PSR-16 is "simpler" than PSR6
* PSR-11: dependency injection container interface
* PSR-7: HTTP messages
* PSR-17: HTTP factory interface (to make creating PSR7 objects independent of the framework)
* PSR-18: HTTP client interface
* PSR-15: HTTP request handler and middleware interfaces
* PSR-14: event managing (draft!)
* PSR-8: huggable 😂
