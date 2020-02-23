---
layout: post
title: "Reading *paid* Financial Times articles for free"
date: 2020-02-22 17:34:14 -0700
categories: jekyll update
---

**Note:** FT disabled the app so you cannot read the news for free anymore.
You can still access the archived versions though, with the link provided at
the end of this article. This incident happened back in August 2018.

![ft on medium]({{ site.url }}/_assets/images/ft_medium.png)

#### Paid Financial Times ... for free

I once stumbled upon an article from Financial Times, an online newspaper
with emphasis on business and economic news.

[The article](https://medium.com/ft-product-technology/making-a-request-to-the-financial-times-b2119a2f422d),
named *What happens when you visit FT.com?*, describes what happens in the background
when one makes a request to [ft.com](https://www.ft.com). Besides many interesting
technical details, I noted that FT used [Heroku](https://www.heroku.com) for hosting its services.

If you have ever used Heroku, you know that apps get public `<NAME>.herokuapp.com` domain by default.
For example, say I register a Heroku app and name it **apple**. When I deploy my **apple** app (e.g.,
a RESTful web server), I will be able to access it on `https://apple.herokuapp.com`.

The moment I saw a word *Heroku* in their article, I wondered if I could randomly
guess any of their services' URL. Out of curiosity, I tried a few. Surprisingly, one of the URLs (namely, `financialtimes.herokuapp.com`)
worked; I managed to access the same contents as in `ft.com`.
What was more exciting is that I could access paid articles **for free**.
In other words, one could read all articles, paid or not, without any subscription whatsoever.

![reporting the issue]({{ site.url }}/_assets/images/ft_bypass.png)

Being not sure how serious this issue is, I nevertheless hoped to get some
bounty for finding out a "backdoor" that let anyone read paid articles for free
(in other words, authorization could be bypassed). After reporting the issue,
even though I got nothing back (\*sigh\*), the URL link now does not render any FT
content and the issue seem to be resolved.

It may have happened that one or more engineering staff members forgot to remove that app. Who knows,
maybe it was used for testing (e.g., black-box system testing) or something else.

**P.S.** You can access archived versions of the website from 2017 [at
archive.org](https://web.archive.org/web/20171101000000*/http://financialtimes.herokuapp.com/).

