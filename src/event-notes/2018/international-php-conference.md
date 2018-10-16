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
