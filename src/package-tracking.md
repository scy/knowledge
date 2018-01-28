# Package Tracking

Some information on package or mail tracking services and APIs.

## Deutsche Post

### Einschreiben (Registered Mail)

Currently (2018-01-28) the tracking web form uses POST requests, so it’s not immediately obvious how to create a tracking URL for registered mail. 
However, it’s possible.

Example link: `https://www.deutschepost.de/sendung/simpleQueryResult.html?form.sendungsnummer=RG083025025DE&form.einlieferungsdatum_tag=23&form.einlieferungsdatum_monat=10&form.einlieferungsdatum_jahr=2012`

Replace `RG083025025DE` with the tracking number (“Sendungsnummer”), `23` with the day, `10` with the month and `2012` with the year when the Einschreiben was brought to the post office, i.e. the date of the receipt. 
Thanks to the JTL Forum for this information. 
([Source thread](https://forum.jtl-software.de/threads/tracking-url-deutsche-post.13981/))
