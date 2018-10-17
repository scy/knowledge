# International PHP Conference 2018

October 15 to 19, Munich; [phpconference.com](https://phpconference.com/)

These are some notes I took during the talks. They don't necessarily contain everything the presentation was about, but what I personally found noteworthy. I didn't check facts and figures for accuracy. I might provide sources to some statistics or whatever at a later point.

## [Is a new Cross-platform Development Era coming?](https://phpconference.com/web-development/is-a-new-cross-platform-development-era-coming/)

by Maxim Salnikov ([@webmaxru](https://twitter.com/webmaxru)), [recording on YouTube](https://youtu.be/QMyo7UVfg_A?t=868)

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

## [WebAuthn: Passwords are Legacy](https://phpconference.com/performance-security/webauthn-passwords-are-legacy/)

by Arne Blankerts

* (a lot of introduction)
* HTML5 actually specifies a [`<keygen>` element](https://en.wikipedia.org/wiki/SPKAC) to generate client side certificates
* (more stuff that's not about WebAuthn either, but about why passwords suck)
* WebAuthn is an official W3C standard, a sub-spec of FIDO2
* "public keys for authentication in browsers"
* supported in Firefox 60+, Chrome 65+ and Edge starting in October
* needs JavaScript to work

## [Zend\Expressive 3 – The Next Generation](https://phpconference.com/php-development/zendexpressive-3-the-next-generation/)

by Ralf Eggert

* based on PSRs, PSR-7 was the kick-off for the Expressive project, ZE2 focused on PSR-11, ZE3 on PSR-15
* ZE3 now has components for sessions, CSRF protection and flash messages
* (talk focuses on differences between the ZE versions and the migration paths, I was hoping to get at least some kind of introduction since I'm new to the framework)
* check out: [Swoole](https://www.swoole.co.uk/), PHP C extension that provides coroutines for PHP
* Expressive is the future, Zend Framework has lost momentum

## [Metamorphosis: From Database-Driven to DDD](https://phpconference.com/web-development/metamorphosis-from-database-driven-to-ddd/)

by Julie Lerman, [recording on YouTube](https://youtu.be/ndA-00usnS4?t=651)

* in DBase, you built databases and put a form on top of it
* even when using .NET, she was focusing on the data instead of the business domain
* path to her transformation
  * ALT.NET vs Entity Framework
  * start with POCOs
  * automated tests (TDD came later)
  * Eric Evan's DDD book
* the book starts not about programming, but about communicating with clients, about understanding their domain
* in DDD, it's more about the business logic, and storing and retrieving data becomes more of a secondary concern
* DDD helps you solving the messy complex things by breaking them down into small, solvable, contained and interconnected problems
* the book even helps the more "introverted" people by walking them through client conversations
* strategic design: analyze and model the business problem, identify the best strategies for solving the problem
* bounded context (against leaky abstractions)
* ubiquitous language isn't cross-context
  * use the same words from the time when talking to the client all the way down to the code
* duplication can be hard to accept, but is required so that changes in one bounded context don't mess up another
* tactical design: …
* reducing side-effects of relationships
  * one-way by default
  * value objects by default vs 1:1 relationships
* avoid overkill: not everything needs to be modeled with DDD patterns
  * if the application is just data in and data out, that's fine as well; keep things simple

## [Angular, React, Vue and Co. – peacefully united thanks to Web Components and Micro Apps](https://phpconference.com/web-architecture/angular-react-vue-and-co-peacefully-united-thanks-to-web-components-and-micro-apps/)

by Manfred Steyer ([@ManfredSteyer](https://twitter.com/ManfredSteyer)), [recording on YouTube](https://youtu.be/kGwNemiInIw?t=549)

* web components
  * framework independent
  * there are several standards for them
    * templates
    * HTML imports
      * forget about them, the browser vendors won't import them because there's already ES imports
    * custom HTML elements
    * shadow DOM
  * can be polyfilled down to IE11
  * simply extend `HTMLElement` and use `attachShadow` and `shadowRoot`
  * doesn't influence the main DOM
  * Angular has "Elements" for custom HTML elements, Vue.js has a CLI switch, in React you have to do this by hand
* micro apps aka micro frontends
  * basically what's called "microservices" in the backend
  * increase maintainability, reduce communication overhead
  * flexible to choose architechtures and frameworks for the apps instead of having to use one for everything
  * pros: separate deployment, mix different technologies, less coordination, less complexity
  * cons: distributed system, distributed data, UI composition
  * UI composition is the important thing to solve
    * hyperlinks between several tiny single-page applications?
      * sounds cheap, but e.g. Google does this: there's the menu to switch between SPAs (Gmail, Maps etc.)
      * simple, but you're losing state, and it's hard to have a consistent UI between them
    * (SPA based) shell
      * ugliest way to do this: iframes (but: provides the best amount of isolation!)
      * bootstrapping several SPAs in one `index.html`
        * in this case, consider wrapping the apps into web components
* choosing a solution
  * do you have shared state and a lot of navigation between applications?
    * little: use hyperlinks
    * much: do you have to integrate legacy apps (PHP, Java) or need very strong isolation?
      * yes: iframes (not so awesome for public websites, you'll not win an award for this though)
      * no: do you need separate deployments or mix technologies?
        * yes: try web components to bootstrap several frameworks as web components
        * no: go with a monolith and libs in a monorepo
* conclusion: web components allow decoupling from your framework, micro apps allow decoupling teams and projects

## [A Journey Into Strategic Domain Design](https://phpconference.com/agile-devops/a-journey-into-strategic-domain-design/)

by Leandro Lages ([@leandrolages](https://twitter.com/leandrolages))

* Why DDD?
  * over time without care and consideration, software turns into a ball of mud
  * it's not only an engineering problem, but also of communication and company structure
  * if your domain is already complex, your software usually adds complexity to it
  * how can we decouple this into smaller parts?
  * problem space: domain, subdomains
  * solution space: context map, bounded contexts
  * the language between these should be common: ubiquitous language
  * tactical patterns: entities, value objects, repositories, factories (mainly code)
  * strategic patterns: focus on organization structure and technical aspects; not limited to source code
* Context Mapping
  * reality map with contexts and models in them
  * legacy system can be viewed as another context in that map
  * when you notice that the language is changing, you're most likely in another bounded context
    * (this is before ubiquitous language)
  * you can also draw maps for organizational and technical reality
  * focus on the domain, not on the data, even if you already have technical realities (certain databases etc.)
  * all bounded contexts usually have upstream and downstream
* Strategic Patterns
  * shared kernel
    * e.g. employee model is common to payroll and HR contexts
    * but usually in the same subdomain
  * anticorruption layer
    * before a model comes into my context, it will go through a layer that does the translation into my language
    * instead of using another context's language
  * open host service
    * on top of anticorruption layer (ACL)
    * if you use the same ACL between several contexts to a common upstream, instead try to provide a service/translation layer at the upstream
    * downstreams don't need to implement ACLs anymore
  * customer/supplier
  * conformist
    * similar to customer/supplier, but no communication
    * usually used if the ACL is expensive to implement
    * downstreams simply use what upstreams provide
    * bad if you have it _inside_ of your organization
  * separate ways
    * no relation between contexts at all
    * instead of using a model from another context, you're building your own model
    * starting point when experimenting (no need to use data from another context, just fake your own)
  * partnerships
    * two teams work together to create a common context without upstream/downstream relation between them
    * usually to create a shared kernel
* communicating the context map
  * don't try to find a tool for drawing your context map, just draw on paper or something
  * you don't need to include _all_ of your company, focus on the important parts
* never think of the data first!
* example: Get Your Guide
  * (I'm not noting down all the details of the example)
  * trying to create teams on top of _domains_
  * strategic importance of context maps:
    * retaining integrity (nobody changes your model in a context)
    * a plan for attack, how to distribute team
    * understanding ownership and responsibility
    * revealing areas of confusion in business workflow
    * identifying nontechnical obstacles (e.g. team structure)
    * encourages good communication
    * helps on-board new starters
  * DDD is not only for engineers!
    * you need to involve domain experts, UX people, etc.
    * event storming is a good method for aligning the company
* suggestions:
  * book: Patterns, Principles and Practices of Domain-Driven Design (Scott Millett / Nick Tune)
    * check YouTube for talks by Tune
* developers might be sceptical: where's a successful example?
  * Microsoft is using DDD heavily
