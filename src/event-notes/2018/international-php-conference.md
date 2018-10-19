# International PHP Conference 2018

October 15 to 19, Munich; [phpconference.com](https://phpconference.com/)

These are some notes I took during the talks. They don't necessarily contain everything the presentation was about, but what I personally found noteworthy. I didn't check facts and figures for accuracy. I might provide sources to some statistics, links to slides or whatever at a later point.

## My personal tl;dr

The things I found most interesting or enlightening or relevant to my current work. The "from" links point to the respective talk's section on this page.

* Having PhpStorm's code inspections is nice, but there are also awesome **static analysis** tools like Psalm available to integrate into your CI pipeline. ([from](#squash-bugs-with-static-analysis))
* Instead of using a cache to speed up page generation, why don't you transform your site into a **push architecture** by pre-rendering everything, even personalized results, based on events? ([from](#performance-in-a-personalized-world))
* **Self-governing teams** can solve organizational and speed issues for startups that grow. Involve people, but don't be forced to wait for them. ([from](#working-in-autonomous-teams-why--how))
* Not accidentally fucking up **cryptography in PHP** is finally getting easy, because libsodium is now bundled with PHP 7.2. Previous versions can get it from PECL or even use a PHP implementation. Use Argon2id for password hashing. SHA-512 is faster than SHA-256 (on 64-bit processors). ([from](#crypto-for-everyone--libsodium-in-php-72))
* Check out **Domain-Driven Development**. The concept isn't new, but by focusing on the _problem_ instead of the solution, it helps solve some of the complexity that software projects often have, and it helps you to better understand your users and stakeholders. ([from](#metamorphosis-from-database-driven-to-ddd))
  * Of course there are also **patterns** to help you make the right decisions, both while understanding the problem and while solving it. ([from](#a-journey-into-strategic-domain-design))
* **Pair programming** is actually faster than programming alone _and_ produces less bugs. **Code reviews** should regularly also be done as a team. And your **coding styleguide** should focus on things that can _not_ be validated by tools. ([from](#effective-code-reviews))
* There's actually a name for modeling your data to not keep _state,_ but all of the _history_ and derive the state from it. It's called **event sourcing** and there are tools for it. Also, it plays nice with one of today's buzzwords, **CQRS**. (And DDD.) ([from](#ddd-event-sourcing-and-cqrs--theory-and-practice))
* **View models** are a really exciting concept to make your view code (and only it, not even including the template engine) unit-testable. Check out [Templado](https://templado.io/) for a templating engine based on this idea. ([from](#do-you-verify-your-views))
  * If that's too radical for you, at least use techniques like service layers and dependency injection to **decouple your code** for easier testing. ([from](#asserttrueisdecoupledmy-tests))
* [whatwebcando.today](https://whatwebcando.today/) and [PWA Stats](https://www.pwastats.com/) are awesome resources if you think about writing a **progressive web application** (PWA). ([from](#is-a-new-cross-platform-development-era-coming))
  * And **web components** help you structure it or even integrate legacy frontends. ([from](#angular-react-vue-and-co--peacefully-united-thanks-to-web-components-and-micro-apps))
* Be really careful when running **shell** commands from within PHP. ([from](#tales-from-the-wrong-end--a-maintainers-story-of-open-source--cves))
* **WebAuthn** is a new technology for logging users in without a password. It's not quite there yet, but worth watching. ([from](#webauthn-passwords-are-legacy))

## [Is a new Cross-platform Development Era coming?](https://phpconference.com/web-development/is-a-new-cross-platform-development-era-coming/)

by Maxim Salnikov ([@webmaxru](https://twitter.com/webmaxru)), [recording](https://youtu.be/QMyo7UVfg_A?t=868), [slides](https://slides.com/webmax/pwa-ijs-2018/)

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

by Arne Blankerts ([@arneblankerts](https://twitter.com/arneblankerts)) and Sebastian Bergmann ([@s_bergmann](https://twitter.com/s_bergmann))

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

## [Performance in a Personalized World](https://phpconference.com/performance-security/performance-in-a-personalized-world/)

by Stefan Priebsch ([@spriebsch](https://twitter.com/spriebsch))

* interesting words from the introduction: edge site includes
* I didn't take notes during the introduction
* let's build a new frontend that builds a page based on some snippets of data
* these snippets are created by components
* a component publishes content in reaction to an event (push on "publish", change in price etc.)
* now we have a push model
* what if some of the snippets aren't there?
  * if the frontend dynamically asks a component to generate the data, we can't generate response times on cache misses anymore
  * it also makes the system way more complicated
  * so we don't do that
  * we just _define_ that the snippets are always there
* where do we store these snippets?
  * file system? doesn't work good with parts running on different servers
  * key-value store: works if you don't have to do "queries" like "all items for user X"
  * search engine?
* implementing this kind of push architecture can be hard because of organizational structures
* instead of personalizing to a single user, why don't you personalize to personas?
  * many organizations don't even have enough data for per-user personalization
  * doesn't stop you from personalizing further over time
* this "new frontend" can of course also proxy the legacy system
  * the frontend can even replace parts of the legacy page with something from the new push architecture → personalized legacy pages!

## [Squash Bugs with Static Analysis](https://phpconference.com/php-development/squash-bugs-with-static-analysis/)

by Dave Liddament ([@DaveLiddament](https://twitter.com/daveliddament))

* statis analysis can only tell you that your code is incorrect
* tests tell you that a particular scenario is working correctly
* CI and static analysis helps you to reduce the costs of bugs by moving their detection earlier, at least before roll-out
* tool suggestions
  * `jakub-onderka/php-parallel-lint`
  * `composer validate`
  * `friendsofsymfony/php-cs-fixer`
  * `jakub-onderka/php-var-dump-check`
  * `sensiolabs/security-checker`
  * https://github.com/exakat/php-static-analysis-tools
* four types of bugs
  * bug
  * deferred bug: things that work until you put invalid/unexpected/future values in there
  * evolvability defect: roughly equivalent to "technical debt"
  * (false positives)
    * you can also rewrite your code so that static analysis doesn't trigger, it's often even clearer that way
* "do you really expect me to fix the 3183 bugs I currently have in the code?"
  * no, but set these as your baseline
* static analysis in real time can reduce your bug cost to 0 since you fix it before even writing it
* but the advanced checks from the IDE are not running in CI
* other tools: Psalm, Phan, PHPStan
  * all have levels (strictest to least strict)
  * "generics" (e.g. `@return Employee[]` instead of just `array`), even syntax `@return array<string,Employee>` to specify key type
    * key/value syntax currently not understood in PhpStorm?
    * you can do `@return Employee[]` _and_ `@psalm-return array<string,Employee>`
    * PSR-5 is being resurrected and will probably advocate that syntax too
  * ignoring violations
* reducing the number of bugs
  * focus on your business logic (strict), not the framework (less strict or not at all)
  * create wrapper code for calling "faulty" 3rd party code
* `@psalm-param class-string $name` can for example make sure that a valid class name is passed in
* `@psalm-assert !null $expression` means "this method will assert that `$expression` is not null"
* you can use stubs to add annotations to 3rd party libraries
* Dave will soon publish [sarb](https://github.com/DaveLiddament/sarb), which can create a "baseline" of all problems at one point in time and only shows new one you introduced
* Psalm also supports `@template` for "real" generics
* see also
  * https://medium.com/vimeo-engineering-blog/fixing-code-that-aint-broken-a99e05998c24
  * CircleCI (1500 minutes per month for free)

## [Event-Sourcing vs CRUD](https://phpconference.com/web-architecture/event-sourcing-vs-crud/)

by Golo Roden ([@goloroden](https://twitter.com/goloroden))

* CRUD is so familiar to us that we don't think about it anymore
* an update loses previous value, a delete loses all values; they are destructive actions
* it's hard to answer questions about the past
* event sourcing limits us to create and read; we add to the list of events, but we never remove from it or edit it
* simple example: bank account with income and expense events
  * in order to get the balance, all relevant changes have to be _replayed_
* allows us to in the future answer questions about things that happened in the past
* you can do _snapshots_ in order to not have to replay everything from the beginning
* pros: (some obvious ones), semantic expressiveness: your stored events state the _intention_, not only the result
* cons: (obvious ones), hard to check for uniqueness, GDPR
* solving GDPR issues
  * simple idea: don't store anything related to humans directly in the events; instead, store it in another system and just refer to it
* if your domain isn't "story-telling" and the history isn't relevant, you don't need to use event sourcing

## [AssertTrue(isDecoupled(“my tests”))](https://phpconference.com/testing-quality/asserttrueisdecoupledmy-tests/)

by Dave Liddament ([@DaveLiddament](https://twitter.com/daveliddament))

* we want to reduce development and maintenance costs of the test suite
* value of tests = (cost of bugs found by suite) - (cost of suite)
* sometimes you do a small change and half of the test suite fails
* _coupling_ is the degree to which two objects know about each other
  * ideally, object A only knows about object B's interface
  * if it knows internal details, changes in B may force us to change A as well
  * the more loosely coupled, the easier it is to create a test double for B
* first example
  * automated testing via Selenium
  * request "can we change the layout of page X"? now half of his Selenium tests failed, because they relied on the interface, even if they weren't _testing_ it
  * UI changes often
  * reduce coupling to the UI!
  * idea: page object translates high-level request ("login") to actual UI steps ("find text box, click here, enter this, …)
    * changes in the UI only require the page object to be updated
  * request: can we change the page a user goes to after logging in?
  * idea: introduce a DSL layer with high-level things like"log in", "get score" and talks to correct page objects
    * tests now contain `assignUserToTeam`, `answerQuestion` etc. and are more decoupled and actually easier to read
  * lessons:
    * testing business logic via UI is difficult, time consuming and brittle
    * introduce layers between tests and software under test
  * but what happens if we replace the entire site with an app or an API?
* second example
  * layered architecture: you want to test the core, the business logic
  * put a service layer around
    * the business logic doesn't even know whether it's a web application
  * the core of course has to know interfaces to things like the payment service or email gateway (e.g. via adaptor)
  * you might even get rid of the DSL and talk directly to the service layer from the tests
  * what do we test at UI level?
    * maybe you don't even need automated UI tests anymore
  * summary:
    * testing business logic at integration level is much easier
    * we need to architect our code to make this possible
    * but this has other benefits as well
* third example
  * we sell our service to different companies, so login now requires username, password and subdomain
  * tests were putting data directly into the database, this now failed
  * db should be treated like another third-party service
  * idea: object mother pattern for users
  * also: builder pattern (creates user with default values set, you can change to only what you need)
  * repository pattern for db?

## [Effective Code Reviews](https://phpconference.com/testing-quality/effective-code-reviews/)

by Frank Sons ([@FrankS](https://twitter.com/FrankS))

> Peer code reviews are the single biggest thing you can do to improve your code.
> – Jeff Atwood

* Why are you doing code reviews? What are you looking for? Can you answer these questions?
  * cf [Microsoft survey](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/ICSE202013-codereview.pdf)
* Are you tracking your results? (Fixing right now or later? Different opinions? How successful are your reviews?)
  * different opinions: Asking a third person to decide may not be helpful (cf Joel Spolsky: "I am the person with the least knowledge about the problem, I won't decide for you")
* results depend on experience, timing and motivation of the reviewing dev
* Let's make it effective.
* mindset
  * Leave your ego at the door. If nobody will tell you how bad you are, how can you improve?
  * There's _always_ something that can be improved.
  * Forget about "your" code. Collective ownership. Share the knowledge. The team is responsible.
    * If you reviewed code, you should be knowledgeable enough to fix bugs in it later on.
* Maybe don't do only pull request reviews. Large pull requests will be left lying around the longest.
  * try team reviews
  * no longer than 90 minutes
  * maybe one review meeting per 2-week sprint
  * don't pick random code, but something that's _important_
  * don't have the author explain the code: pick someone at random to explain it
    * solves the problem of people not asking questions
  * _understand_ every line, especially complicated ones with formulas or something
* intention
  * if you're writing an experiment, _learnings_ are more important than quality
  * if you're writing a payment system, quality and security are more important
  * pick the right method
    * for style, pull requests are fine
    * for finding the best solution, pair programming is probably better, or team reviews
* create a coding guideline
  * don't add stuff you can check with tools
  * let the team create it
  * it's a living document
  * how are we going to do things? what are the "best" solutions to things? best practices?
* keep track of results
  * make sure you have follow ups, e.g. architectural flaws
  * don't discuss longer than 5 minutes in the review
  * not everything needs to be fixed at once
    * of course, if it just takes you only one minute, do it
* create a review checklist based on the guideline you've created
  * "The only difference between screwing around and science is writing it down."
* reviews are part of the development and shouldn't be second-class citizens
* don't document what the code is doing, but _why_ it's doing that
* "yeah this code is complicated but it's working" is no excuse: other people need to _understand_ it in order to be able to fix bugs in it
* if nodoby can reason _why_ solution A is better than B, it's opinion
  * otoh, if there is a reason why one of these is better, choose that one
  * try to not care about things that are opinion
  * for opinion things, vote as a team, or in the worst case, flip a coin
* there are studies showing that pair programming is faster
  * in the beginning it's slower but produces far less bugs
  * later on it's even faster than doing it on your own _and_ produces less bugs
  * see Johann Peter Hartmann's [Management Brainfucks](https://de.slideshare.net/johannhartmann/management-brainfucks)
* try mob programming at some time, but it's not something to do regularly

> I've seen companies where they don't put all of the developers on the same plane, because if it crashed, the company would go bankrupt.
> – Frank Sons

## [DDD, event sourcing and CQRS – theory and practice](https://phpconference.com/web-architecture/ddd-event-sourcing-and-cqrs-theory-and-practice/)

by Golo Roden ([@goloroden](https://twitter.com/goloroden))

* example he used: [Never Completed Game](https://www.nevercompletedgame.com/)
* the hard thing is not to create the game, but to come up with the questions
  * experts do that
* language is a core thing: open game, start game, create game?
* a command will either be fulfilled or denied and cause events
* you can have the discussion about how to name things with anyone on the team, there's no technology involved
* in DDD, a thing that has a state and reacts to commands is called an aggregate
* a word ("game") can mean different things to different people
  * DDD solves this by introducing a bounded context, and in that context the word has a single, well-defined meaning
* so the context has a ubiquitous language and one or more contexts
* event sourcing can be done without DDD, but they play nice with each other, since DDD has events, too
* CQRS is CQS on the application level
* CQS means that each method is either a command or a query
  * this may seem easy, but think of `stack.pop()`
* so CQRS splits for example an API into a read and a write API
* joins are expensive, reading is important: 5NF databases are not always the goal
* have a write and a read database
  * can now be scaled independently
* writes to the write DB are pushed to the read DB using a message queue, which of course causes eventual consistency

## [Tales from the wrong end – a maintainer’s story of open source & CVEs](https://phpconference.com/php-development/tales-from-the-wrong-end-a-maintainers-story-of-open-source-cves/)

by Marcus Bointon ([@SynchroM](https://twitter.com/SynchroM))

* presenter is maintainer (not original author) of PHPMailer, really popular package existing since 2001
* once upon a time: [security issue in PHPMailer](https://github.com/PHPMailer/PHPMailer/issues/903)
* coordinated disclosure:
  * someone finds a vulnerability, reports it to vendor
  * vendor develops and releases patch
  * vulnerability disclosed
* PHPMailer: CVE-2016-10033, CVE-2016-10045
* CVE type alone doesn't tell you a lot, even if it's a remote code execution
* there's an additional severity rating
  * the PHPMailer issues were rated 9.8/critical
* CVE-2016-10033
  * `$params = sprintf('-f%s', $this->Sender)`
  * attack string can be a valid email address! so we escape it, right?
  * `$params = sprintf('-f%s', escapeshellarg($this->Sender))`
  * before release, someone posted an exploit → we had become a zero-day
  * so fix was rushed out as 5.2.18
    * also known as CVE-2016-10045
* CVE-2016-10045
  * we think we're doing `$mailcommand . escapeshellarg($param)`
  * but `mail()` applies `escapeshellcmd()` internally
  * `escapeshellcmd($command . escapeshellarg($param))`
  * result is undefined and exploitable
  * workaround released in PHPMailer 5.2.20
  * this is fundamentally a PHP bug
  * we ended up checking whether the escaped version is different than the non-escaped version and if it is, we refuse to use it
* affected Wordpress, Joomla, Drupal etc; articles in press
* but comments on handling the vuln were generally positive
* how did other mailers fix this?
  * they didn't!
  * Zend_Mail, SwiftMailer, RoundCube all vulnerable
  * similar bugs in Python, Ruby, Node
* researcher wrote a long article about the general vulnerability of PHP's `mail()` function
* lessons learned
  * don't use `mail()`, use SMTP to localhost
  * open source is awesome
  * when it matters, people will show up to help, it's not all left to you
  * security researchers and whistleblowers need effective legal protection as a matter of national policy
* donations to open source?
  * there's a lot of time, effort and enthusiasm going into it
  * differentiate between project and personal
    * PHPMailer could use a code audit
  * Patreon is great for writers
    * open source maintainers will almost need to _become_ a writer or regular blogger to benefit from it
  * PayPal is probably better
  * make it easy!
  * his experience: one 0.25 EUR donation, another one with 0.75 EUR and after mentioning it in a talk another one: 9 EUR so that he is now at a nice round 10
* thanks to
  * security researches - we need them!
  * those who comment, submit PRs, review code
  * everyone providing feedback and bug reports
  * those that answer questions on Stack Overflow

## [Working in Autonomous Teams: Why & How](https://phpconference.com/agile-devops/working-in-autonomous-teams-why-how/)

by Tina Dreimann ([@tina_3men](https://twitter.com/tina_3men))

My notes don't reflect how inspiring this talk was. At some point, I even stopped taking notes and just listened. I'm afraid that even once the slides are available, they won't give a lot more insight. Talk to Tina if you can, she has a lot of experience and new solutions.

> Control leads to compliance; autonomy leads to engagement.
> – Daniel Pink

* often autonomy is mistaken for anarchy
* they work in squads and tribes
* main reason for that was growth
* they were in a place were they were slowed down by their size
* CTO: "I was looking so long for ways to motivate people, and then I realized that I simply have to get out of their way and not _demotivate_ them."
* diversity: every important role in one team; don't lock developers away in a room
* growth mindset is everything
* every team needs their missing & vision
  * mission should be the north star, never be reachable
  * vision should be a clear playground, a picture of the "preferred future"
* self-governing teams set their own objectives & key results
  * OKRs: ambitious, focused, team-driven, fully integrated
  * useless if it's not defined by the team together
  * don't just name a person to do it, you're missing the diversity!
  * at least have a team discussion
* advice processes are not optional
  * before you make a decision, get advice from someone
  * (even in an autonomous team!)
* decision making is more than yes or no
  * difference between consent and consensus
  * not everyone needs to say yes, they just need to not object
  * else it slows you down
* squad size
  * if you focus on the business priorities, you will be able to do a natural split
  * try slicing by customer groups
  * in the beginning they started with fully-stacked teams
  * teams got too big
  * give the teams the time to get rid of legacy!
* put people into the same team to make them understand each other better (e.g. devs/marketing)

## [Crypto for Everyone – Libsodium in PHP 7.2](https://phpconference.com/php-development/crypto-for-everyone-libsodium-in-php-7-2/)

by Marcus Bointon ([@SynchroM](https://twitter.com/SynchroM))

> If you type `mcrypt` in your editor, stop. You're doing it wrong.
> – Marcus Bointon

* crypto functions by number of keys involved
  * 0 keys: hashes, PRNGs, key derivation
  * 1 key: MACs, secret key encryption
  * 2 keys: key exchange, public key encryption, digital signatures
* core & extension crypto in PHP: mhash, mcrypt, openssl, pecl-gnupg, hash, pecl-scrypt, password_hash, hash_equals, CSPRNG, pecl-libsodium, sodium
* so sodium came into PHP 7.2 after being available as a PECL extension since 5.6
* OpenSSL is okay, depending on which part of it you're using
* libraries: zend-crypt is okay, phpseclib is based on mcrypt, don't use it
  * sodium_compat is libsodium reimplemented in PHP
  * ciphersweet allows searches on encrypted data
* libsodium is a fork of NaCl, supported on more platforms (also in JS and WASM), multiple language bindings, audited code
* NaCl is by DJB, Tanja Lange etc., libsodium by Frank Denis (pure-ftpd), Scott Arciszewski
* sodium takes away choices so that you can't choose the wrong thing
* side channel attacks: timing, thermal, RF emissions, light, sound power (Spectre/Meltdown, password hash timing, Ethernet switch LEDs connected to the data lines)
  * instead of `WHERE email='…' AND password='…'`, do the comparison in PHP, but use `password_verify` instead of a string comparison, which will return early on the first character mismatch
  * or `sodium_crypto_pwhash_str_verify`
* SHA-512 is more efficient on a 64-bit processor than SHA-256
* Argon2i is resistant against timing attacks, Argon2d against GPUs/parallelization, Argon2id against both
* when writing new apps, use the best hash currently available (rehash on login to upgrade)
* hashing with sodium
  * `sodium_crypto_shorthash` for verification and non-crypto purposes
  * `sodium_crypto_generichash` (uses e.g. BLAKE2b)
  * `sodium_crypto_pwhash_str` for passwords (e.g. Argon2id)
* message authentication codes
  * `sodium_crypto_auth`, `sodium_crypto_auth`
* the CSPRNG in PHP7 is good, which is why sodium doesn't replace it
* secret-key encryption
  * `sodium_crypto_secretbox_*` for combined encrypt-then-MAC
  * `sodium_crypto_aead_*` (authenticated encryption with associated data)
  * ChaCha20-Poly1305 is efficient on low-power machines, AES256-GCM is basically less efficient, except on most modern processors since it's hardware-accelerated
* public-key encryption
  * `sodium_crypto_box_*` functions
* key derivation: `sodium_crypto_pwhash`
* key exchange: `sodium_crypto_kx_*`
* `sodium_memzero` to clear the key etc. from memory
* `password_hash` in 7.2 supports Argon2i, but requires libargon2, which isn't included by default; will be fixed in 7.3
