---
layout: post
title: "PuppetConf 2013: Advanced Puppet Training"
date: 2013-08-21
tags:
  - PuppetConf 2013
  - PuppetConf
  - Puppet
comments: true
redirect_from: /2013/08/21/puppetconf-2013-advanced-puppet-training/
---

So, it’s Wednesday morning and I’ve been in San Francisco since Sunday for PuppetConf. The conference doesn’t start until tomorrow (Thursday) but I opted to attend Advanced Puppet training which came with free admission to the conference. I’ve been using Puppet since 2009 and I’ve never attended any kind of training for it. I’ve read all the books, most of the documentation, and tons of blogs and other resources. I didn’t have particularly high expectations for the training; I guess I was looking to use it to fill in some gaps. I’m sorry to say that the class hasn’t been able to live up to my already low expectations.

The instructor is very nice and seems generally knowledgeable, but he had a hard time explaining some of the more nuanced topics like [anchors](http://docs.puppetlabs.com/puppet/2.7/reference/lang_containment.html#workaround-the-anchor-pattern) and [inheritance](http://docs.puppetlabs.com/puppet/3/reference/lang_classes.html#inheritance). He also had a hard time pacing the class. The class is meant to be a three day class; the materials are split up into days, and they aren’t particularly ambitious as far as material coverage goes. Here’s the agenda according to the training guide:

Day 1:

- Puppet Basics Review
- Facts and Functions
- Classification
- Advanced Coding Techniques
- Roles and Profiles
- Best Practices

Day 2:

- File Manipulation
- Data Separation
- Virtual Resources
- Exported Resources and Collections
- Server Scaling

Day 3:

- Advanced Reporting
- Troubleshooting
- Provisioning
- MCollective

Now, I’m not convinced all of these topics are appropriate for an “advanced” level class, and I’ll talk more about that–but by the end of Day 1 we had barely gotten through the “Facts and Functions” section. I knew something was up when, by the 2pm break we hadn’t finished the “Puppet Basics Review” section. Yesterday, by the end of the day, we had barely finished the “Data Separation” section. Class starts in about 30 minutes, and I can’t imagine we’re going to get through the last seven sections by the end of the day. What really bugs me is that most of the really “advanced” content is on the back end of the class! He spent WAY too much time on the fundamentals. I was talking to some of the trainers at lunch yesterday and most of them said that they were in about the same place with their advanced classes.

I don’t get it. The Puppet training website clearly lists “Puppet Fundamentals” or “equivalent hands-on experience with Puppet” as a prerequisite for the Advanced course. The “Basics Review” section of the training manual is 33 pages long. For perspective, the “Advanced Coding Techniques” section is 32 pages, “Roles and Profiles” is 30 pages, and “Data Separation” is only 22 pages. I think the folks at Puppet Labs need to sit down and figure out what this “Advanced Puppet” class is meant to be. There’s no mention in this class about rspec, continuous integration, advanced module development, or any number or truly advanced topics necessary to take your Puppet code to the next level.

Now, I knew the agenda when I bought the ticket, so I’m not faulting Puppet for that. I just think that there is a need for a truly advanced level Puppet class. I would like to see a class devoted to coding best practices, test driven module development, continuous integration and deployment of Puppet code, and integration of Vagrant and rspec-system into the module development workflow. These are the kinds of “advanced” topics that are really useful to the community.
