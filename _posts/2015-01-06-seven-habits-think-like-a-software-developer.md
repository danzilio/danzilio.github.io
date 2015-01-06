---
layout: post
title: "The Seven Habits of Highly Effective Puppet Users: Think like a software developer"
date: 2015-01-06
category: Seven Habits
tags:
  - Puppet
  - Seven Habits
comments: true
---

I gave a talk at PuppetConf 2014 entitled [“The Seven Habits of Highly Effective Puppet Users”](https://puppetlabs.com/presentations/seven-habits-highly-effective-puppet-users-david-danzilio-constant-contact) based on a collection of observations that I’ve made over the years regarding high-functioning Puppet users. These users exhibit common behaviors that go far beyond policies and procedures. These behaviors are broadly understood by team members and nearly universally adhered to. It’s clear that these habits lead to more stable, maintainable, and well-understood Puppet deployments. This is the first in a series of blog posts discussing those seven habits.

**Habit 1: Think like a software developer**

By far, the most consequential habit exhibited by highly effective Puppet users is the tendency to think like a software developer. Much to the chagrin of many folks in the field, infrastructure as code has fundamentally changed the practice of system administration. This change is so great that it warrants a redefinition of the job. I hate to be the bearer of bad news, but you’re not a system administrator anymore; you’re a software developer. If you want to keep up with the industry, you’re going to need to re-tool. The sooner you make peace with this fact, the sooner you can start working on taking your skill set (and your Puppet code) to the next level.

I wasn’t the only speaker preaching this at PuppetConf, it was a consistent theme throughout the entire conference. If you want to really put Puppet to work, you need to move beyond package-file-service and resource declarations. You need to approach Puppet from the perspective of a software developer. Puppet’s strengths are in its extensibility and flexibility. If you’re still approaching Puppet as if it’s just a collection of data, you’re never going to really harness Puppet’s core power. If you’re writing Puppet, you’re writing software. I know, it’s very meta and can be hard for some people to wrap their minds around, but the first step is to realize that Puppet is a real programming language.

Puppet is a [Domain Specific Language (DSL)](http://en.wikipedia.org/wiki/Domain-specific_language) and, consequently, is missing a lot of the features of a general-purpose language. This isn’t a flaw; it’s a design decision. A DSL allows the user to focus on the problem domain instead of the language. The whole idea of a DSL is that language is an impediment; we need to get the language out of the way of the real work. We may have gone a little too far with this. We thought Puppet’s DSL would allow system administrators to write good code without having to learn what good code looks like. Instead, all this did was make it easier for them to write really bad code. Part of thinking like a software developer is taking the time to know the language inside and out. Once you shift the focus onto the language a little more, you will completely change your approach to Puppet, for the better.

So, what does this really mean? It means that you’re going to have to do some digging to figure out exactly what a software developer does. You’re going to have to learn the lingo and understand how to do things like:

- continuous integration
- release engineering
- code review
- documentation
- test driven development

These things are fairly foreign to system administrators, but they’re about to become central to your new job description. Luckily, I’m here to help. Over the course of this blog series, I’m going to touch on many of these topics. Stay tuned!
