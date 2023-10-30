---
title: "Who Were You Denvercoder9?"
date: 2023-10-30T20:36:19+01:00
draft: false
---

So if you didn't know the title of this post refers to an xkcd called [Wisdom of the ancients](https://xkcd.com/979/). It's one of my favorite xkcds and the recognition factor is high. Anyone who's done software development for a while will probably agree.

A while back I inherited a Wordpress site that was being hosted in an Azure App Service that put me in one of those Denvercode9 type situations. The site would perform well as long as the cache was hit, which was more or less all of the time. First time page generation was horrible, however, and the site also had some dynamic content that would be more difficult to cache with the tools we had available, so I started investigating why performance was so bad.

Wordpress and PHP is not a platform that I have much experience with and most people that I talked to only had general suggestions about checking the database performance. Increasing the resources available to the database made no difference and googling for Wordpress and performance gives you page after page of SEO spam. There are a few post on Stack Overflow that point out different possible causes, none of which got me any closer to figuring out what was causing the bottleneck.

Eventually I ended up installing [Xdebug](https://xdebug.org/) to do some profiling, analyzing them with [KCacheGrind](https://kcachegrind.github.io/html/Home.html). Comparing with a baseline measured with the site running locally, the results showed that disk IO was the issue. The top time consuming function calls were `php::file_exists`, followed by `php::file_get_contents`. Those two function calls weren't even in the top 20 for the reference. It was most likely latency rather than throughput that was causing problems, Wordpress sites tend to require a very large number of files to be accessed to serve a single request.

Stack overflow does have [this post](https://stackoverflow.com/questions/67141062/horrible-performance-on-azure-app-service-wordpress) where the accepted answer describes the underlying issue, but finding that at the time was like finding a needle in a haystack.
