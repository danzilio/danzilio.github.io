---
layout: post
title: "PuppetConf 2013: Day 1, Part 2"
date: 2013-08-23
tags:
  - PuppetConf 2013
  - PuppetConf
  - Puppet
---

Will Farrington’s presentation [“Puppet at GitHub”](http://sched.co/16C6ffa) was fantastic. It was really interesting to see how a DevOps giant does things. Needless to say, we’re totally intrigued by the idea of [ChatOps](https://speakerdeck.com/jnewland/chatops-at-github). We’re going to start playing with [Hubot](http://hubot.github.com/). Will talked about how they avoid having “hand crafted, artisanal, free range servers” by automating everything from the start. One command to Hubot boots the server using IPMI into memtest for an hour, then into a stress testing regimen for a day, and then into provisioning. We **really** like the sound of this. Will is a fantastic presenter with great stage presence. It’s incredibly clear that the work GitHub is doing is paving the way for innovation in DevOps.

The last session of the day was [“Building Data-Driven Infrastructure with Puppet”](http://sched.co/1bjQ7lB) by James Fryman (also of GitHub). James’s presentation was incredibly well done; he made a great case for treating Puppet, and operations in general, like software development. I think he may have coined the term BeerOps right in front of us (BeerOps is all about spending less time working and more time drinking beer). He spoke very eloquently on the need to provide hooks and interfaces in your Puppet code that allow for more intelligent automation. Once you’ve included these interfaces, it’s significantly easier to take your automation to the next level and allow your systems to respond to events in real time. By making machine-readable inputs and outputs, we can configure our systems to respond to any kind of event however we choose. It was a fantastic talk and I highly recommend watching it when it’s available.