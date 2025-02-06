---
title: "Configuration In dotnet"
date: 2025-02-06T18:53:01+01:00
draft: false
---

Some days that meme that I guess says that the natural evolution of a developer starts with simple code, progresses to get more and more complex, only to sooner or later return to a preference for simplicity. One area that leaves an annoying amount of room for improvising together solutions impossible for mere mortals to wrap their heads around is the way configuration works in .NET.

It used to be, in the old days, that you had to use XML (urg, XML...) and when you wanted to make that XML specific to whatever environment your application was running in, you'd use a bunch of XML-attributes to find tags and replace them, or their values, or their children. At some point or another, ritual sacrifice was probably needed in order to please the XML-gods. The actual transformation was done when compiling the application (or should that be when "publishing" application or does it perhaps depend on if it's an ASP.NET-application or something else?).

In recent versions of .NET, they realized that nobody likes XML and swapped it out for JSON. In order to get environment specific configuration, you typically load JSON-files with the environment name in it. What's the environment name, you ask? Well, it's whatever's in the environment variable ASPNETCORE_ENVIRONMENT. Unless you're a non-web app, in which case it's whatever's in DOTNET_ENVIRONMENT. Also, if your web app is deployed in IIS, the default settings won't propagate environment variables from the host to the actual app.

Once you've loaded your configuration sources (there may be other things than JSON-files), you can access your configuration by binding parts of it to objects, or by indexing into it with magic strings as keys. The former works decently, albeit the boilerplate code get's a bit verbose. The later would in a fair world lead to your laptop providing lethal current across the keyboard. If you really want to, you can use this stringy indexing to map the shape of some JSON into a different shaped object in your application using many, many lines of error prone code.
