---
title: "Large Language Models"
date: 2024-02-18T19:54:51+01:00
draft: false
---

I just read [I worry our Copilot is leaving some passengers behind](https://joshcollinsworth.com/blog/copilot) which (among other things) is about how GitHub CoPilot will suggest code that solves a given problem but is lacking when it comes to aspects other than "it works". In the example, CoPilot insisted on solving a (trivial) problem with JavaScript when the obvious solution only required HTML. The JavaScript solution worked, but wasn't very accessibility compatible and obviously required JavaScript to run in the browser, which was unnecessary.

The content of the post will probably be familiar to you if you write software and tried using LLMs to do so. There have been times when I've reviewed code written with LLM assistance that's had the same type of issues. There is also the part about questionable ethics with regard to how training data is sourced. 'Ethics' is probably not what will stop corporations from adopting a large scale productivity boost to very expensive human labour, but perhaps the other drawbacks are? If a developer uses a deprecated API because it's suggested by an LLM, is the productivity boost worth it when the code needs to be replaced sooner than otherwise needed? Will the "training wheels" provided by LLMs make developers worse at searching for and valuing information? Will LLMs be able to help us make changes to systems that we don't understand? There is definitely an angle where using LLMs for software development doesn't make all that much sense, considering the large proportion of the cost of software that is spent on running and maintaining it.
