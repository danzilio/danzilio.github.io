---
layout: post
title: "The Seven Habits of Highly Effective Puppet Users: Treat Puppet like code"
date: 2015-01-26
category: seven_habits
tags:
  - Puppet
  - Seven Habits
comments: true
---

*This post originally appeared on the [Constant Contact Tech Blog](http://techblog.constantcontact.com/software-development/treat-puppet-like-code/). I'm reposting it here for continuity.*

_**Note:**_ *This is the second post in a [series](http://blog.danzilio.net/tags.html#Seven+Habits) based on a talk I gave at PuppetConf 2014: [The Seven Habits of Highly Effective Puppet Users](https://puppetlabs.com/presentations/seven-habits-highly-effective-puppet-users-david-danzilio-constant-contact).*

Of all the trends in operations, infrastructure as code is a major game changer. Infrastructure as code has brought a lot of new people—many of whom don’t necessarily consider themselves developers—into the software development world. This is, understandably, a strange world for many Ops folks. Traditionally, operations folks have had very different toolchains than their developer compatriots. This has made sense because they were solving different technical problems; the toolchains evolved in separate worlds. Why would an Ops person need to know how to use Git when they’re not writing any code?

Infrastructure as code is just one of the driving forces to unify toolchains between Dev and Ops. We’re now approaching our problems in similar ways—with code. The problem for many Ops folks undergoing this transition is that they don’t necessarily understand what it means to write code; they don’t necessarily know what good code, and good coding practices, look like. A lot of infrastructure code has evolved inside the Ops world and, consequentially, ends up looking very different from traditional code. Many people treat Puppet code like configuration files. It’s important to remember that infrastructure code is real code, and needs to be treated accordingly. OK, but what does that mean? Well, some of the basics include:

- version control
- documentation
- refactoring
- code review
- style

<br />
Let’s take a quick look at each of these fundamental elements of building and maintaining code:

**Version Control**

The first step, actually the zeroth step, is to use version control. Developers use version control because code is their work product. If you’re using Puppet, or any other infracode language, your work product is code too. In the old days our work product was a server—big, beautiful, polished, and one-of-a-kind. Servers were configured by hand and it was hard to tell when things changed. Now, we write code. With code, it’s very easy to tell when things change; to do that, we use version control. There are tons of version control systems out there: [Git](http://git-scm.com/), [Mercurial](http://mercurial.selenic.com/), [Subversion](https://subversion.apache.org/), [Bazaar](http://bazaar.canonical.com/en/), [Perforce](http://www.perforce.com/), [Visual Studio TFS](http://www.visualstudio.com/en-us/products/tfs-overview-vs.aspx), and [AccuRev](http://www.accurev.com/)--just to name a few. There’s bound to be something to satisfy even the most contrarian of users. It doesn’t matter what version control system you use, just be sure to use something. Your infrastructure code is just too important not to check into a version control system.

**Documentation**

Now that you’re writing code, you’re going to have to learn how to write documentation. Back when we were handcrafting servers, we didn’t have a good place to put our documentation. It was a classic coupling problem. Many sysadmin shops have well-neglected Wikis or piles of outdated Word documents on a shared drive somewhere nobody remembers to look. With infrastructure as code, we can put our documentation right inside our source code. Developers document their code because it takes a lot of effort to understand what the code is doing by reading it; they know they’re not the only person who needs to understand the code. Adding new functionality or patching a bug is much easier when you can read documentation instead of having to reverse engineer the code to get context.

Well-documented code has inline documentation in some format that can be parsed and used to generate easy to read source documentation. Puppet has introduced a new tool called [Strings](https://forge.puppetlabs.com/puppetlabs/strings) to help infracoders generate pretty documentation from their code. Strings is built on top of the YARD documentation tool. It allows you to document your code using a variety of different formats.

Documentation best practices include:

1. Inline documentation should fully document all parameters and variables used in classes and defined types.
2. A README should explain how to use a module, what the module affects, the requirements of a module, and any other information necessary to use the module.
3. No new functionality should be added without documentation.
4. Documentation must not be an afterthought; it should be central to your development workflow.
5. You should document your code as you go, because you’re probably not going to do it after the fact.

<br />
**Refactoring**

[Refactoring](http://en.wikipedia.org/wiki/Code_refactoring) is a concept that will be foreign to many sysadmins. Refactoring is a continuous improvement practice where a developer regularly makes small improvements that do not change the external behavior of the code. These changes are geared towards improving readability, reducing complexity, and other improvements that make the code more maintainable or efficient. Refactoring should not change the API or the way the code behaves from a user’s perspective. Refactoring is a central aspect of Test Driven Development, a topic I’ll discuss in another post.

**Code Review**

Code review is also likely to be a fairly novel concept to traditional sysadmins. Before we had a way to express our infrastructure in code, there was no good way to submit your work for comprehensive peer review. Now our code changes can (and should) be reviewed by our peers before they make it into the codebase. This can be enormously beneficial for the code and the developer alike. Code review is also a great tool for communicating amongst team members. It is also a very effective teaching tool. Code review carries many of the same benefits of pair programming—bugs are caught earlier, mistakes (downtime) are avoided, team members learn from each other. There are a number of tools available to aide in this process; the most common tool is [Gerrit](https://code.google.com/p/gerrit/). Code review shouldn’t be painful or disparaging; it’s simply an opportunity to get another pair of eyes on your code and to learn from the experience of others.

**Style**

Now that infrastructure is code, we need to make sure that the code is readable by everyone who needs to work on it. Adhering to a style guide is an important part of this. Coding styles can vary widely from person to person. A style guide helps to ensure that everybody understands the standards that the code is expected to meet. Puppet Labs has a [style guide](https://docs.puppetlabs.com/guides/style_guide.html); you can automate checking your code’s compliance with the style guide using the puppet-lint gem. You can integrate puppet-lint into your development workflow to ensure that it checks your code every time it is committed to a shared repository. You can easily achieve this with your version control system’s hook mechanism or with a Continuous Integration system like Travis CI or Jenkins (I’ll talk more about Continuous Integration in a separate post). Style is substantive when we’re talking about code.

Treating Puppet like code is very important in order to write code that is reusable and easy to maintain. These concepts may be foreign to you, but there are plenty of good examples of this behavior amongst the community. I highly recommend you take a look at the Puppet Approved and Puppet Supported modules on the Puppet Forge for examples of good coding behavior. I’ve talked about some of the basics of treating Puppet like code, but it really is much more involved than what I’ve written here.

Over the course of this blog series, I’ll dig more in depth on what it means to write good Puppet code. Feel free to take a look at my PuppetConf presentation: [Seven Habits of Highly Effective Puppet Users](https://puppetlabs.com/presentations/seven-habits-highly-effective-puppet-users-david-danzilio-constant-contact).

The next post in this series will be on designing for Puppet. Stay tuned!
