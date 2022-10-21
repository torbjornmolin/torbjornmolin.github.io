+++
title = "AutoIt"
description = "Automating Windows GUI apps"
date = 2022-10-21T09:13:50Z
author = "Torbjörn Molin"
+++

![autoit screenshot](/autoit.png)
The last few days I've been running a background type task on a Windows machine that I don't have full access to configure. What I wanted was for the machine not to go to sleep since every time that happened a VPN-connection that I needed would be disconnected and no matter what settings I changed I couldn't get that to happen.

A quick and dirty solution came to mind, what if I just have something automatically move the mouse cursor? I started out googling and there are such utilities available, but then I remembered that I already have AutoIT installed. AutoIt will let you script GUI interaction on Windows and it's quite powerful. The example in the screenshot above is really a "Hello, world"-type example of what AutoIt can do.

I was worried that Windows would be able to make a distinction between actual mouse interaction and programatic interaction from AutoIt and not do what I wanted but it worked perfectly. In the past I've been using AutoIt to interact with dialog boxes in applications without simulating mouse input and that also works very well. The one limitation that I have run into is that AutoIt can't interact with non-native win32 applications. That includes things like WPF applications.
