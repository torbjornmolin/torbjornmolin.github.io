+++
title = "Picking a static site generator"
description = "Why I ended up with Hugo"
date = 2022-08-03T13:13:50Z
author = "Torbjörn Molin"
+++

### A bit of background

I've had this domain for a bit over a decade now and use it for a number of email accounts. Years ago there used to be a Wordpress blog on here about live sound engineering and now that I work as a developer and also recently started freelancing, I felt that now would be a good time to get a new site up.

### Just pick something!

I tried (I really did) not to get too caught up in picking a platform for managing it, but I didn't want to use Wordpress again, it felt clunky and overkill. A static site generator seemed to fit the bill. I first stumbled on [Jekyll](https://jekyllrb.com).

Jekyll seemed good, it _seemed_ like I wouldn't have to do much to get it up and running and that I would have something working in a day or so. But it turned out it was written in Ruby and whatever version of Ruby that comes bundled with macOS is of course not up to date so you need to mess with another version.
So I installed a current version of Ruby, there is a nice guide on the Jekyll webpage. But then there was some other issue and I ended up just generating the site on a Linux machine that I had handy on the network and that worked right away but wasn't very practical.

It reminds me about someone I heard recently complaining about how it's a pain to move python code to another machine and me thinking that shouldn't scripting languages be the exact opposite of that? Anyway, I dropped the project and just left a static HTML page with almost no content in the root folder of the site.

And then today it was raining and so I stayed indoors instead of being outside putting new siding on the summer house and I did a quick search for 'static site generator' again and came across [Hugo](https://gohugo.io). So I did

    brew install hugo
    hugo new site [name of site]

added a theme and then

    hugo serve

and it just worked.

I even took an additional 20 minutes to set up a github action to publish whenever I merge to main and that seems to be working so far.

If I'd just had a working Ruby environment set up on my laptop I probably wouldn't even have looked at Hugo. This is nothing against Jekyll, Jekyll is probably fine for my needs, but I guess it's another aspect to consider when choosing a language to implement in.

Hugo is written in Go, by the way, so no interpreter version to mess with and it's really fast.
