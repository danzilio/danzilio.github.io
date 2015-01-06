---
layout: post
title: "PuppetConf 2013: Day 2 and Beyond"
date: 2013-08-24
tags:
  - PuppetConf 2013
  - PuppetConf
  - Puppet
---

Well, PuppetConf is over. I'm writing this on the plane (with no WiFi...ahem) back to Albuquerque. It turns out that I failed to recognize the nuance between a "direct" and "nonstop" flight, so I'm taking the time to write this blog on the tarmac at LAX. Today began most disturbingly at about 6:40am with my coworker calling my room, rousing me from my deep and wondrous slumber to ask if I intended on making the flight back home. Apparently, another nuance that recently escaped me is the "weekday only" setting on my alarm. But, here I am.

Day 2 of PuppetConf was definitely the stronger of the two in terms of content. I opted to try my hand at the Puppet Certified Professional exam that was being offered gratis by Puppet Labs. The exam was harder than I anticipated, but I passed. I'm now a PCP! (Cue the balloons and champagne.) Also, who came up with the name for this cert? PCP?

I spent the day in the DevOps track (located in the Grand Ballroom). The first presentation after lunch was entitled [“Puppet Module Reusability – What I Learned from Shipping to the Forge"](http://sched.co/15aK64K) by Gareth Rushgrove (UK Government Digital Service). Gareth is a top contributor to the forge and a pretty interesting guy to talk to. His presentation was one of the best I heard at PuppetConf. He’s put together a great tool for anybody writing Puppet code, it’s a skeleton for the puppet module generate command. You can find it at [http://github.com/garethr/puppet-module-skeleton](http://github.com/garethr/puppet-module-skeleton); I’ve been using it for a couple weeks now and highly recommend it. Gareth stressed the need for the community to become more opinionated on coding standards. I appreciate his pythonic approach to code. Gareth also recommended that open source module maintainers insist that contributors write tests for their code before approving a pull request. I couldn’t agree more. I’m not the best at writing tests, but I’m getting better and it’s because I have to—because there is no excuse not to test your modules anymore. He also talked a bit on how tests should be implemented, stressing testing the logic of the module. He stressed that we need to test the interface, not the implementation. We need to go beyond simply rewriting our modules in rspec and pay very close attention to the interfaces we expose in our modules. This is something I need to get better at and I will definitely be using Gareth’s modules as an example of how to do things right.

The next presentation was ["DevOps isn’t Just for WebOps: The Guerrilla’s Guide to Cultural Change"](http://sched.co/11MrQl6) by Michael Stahnke. This was by far the best presentation I saw at PuppetConf and one of the best presentations I’ve ever seen. I highly recommend keeping an eye out for the video of this talk and sending it to everybody you know. I plan on sending it around the office. Michael talked about his experience in a large tech company and how he was able to take it from a complete and utter mess (think The Phoenix Project) to a marginally DevOps oriented workplace. This pattern is very familiar to me; an IT shop where servers are treated as one-of-a-kind works of art and where nobody took an architectural approach to the system. Michael is now an executive at PuppetLabs and his story couldn’t have been more compelling. He came up with five methods for changing the culture in an environment: reduce variability; stop, collaborate, and listen; shout your failures; experimentation matters; and solve causes, not symptoms. His approach makes a lot of sense and is in line with a lot of recent organizational behavior research.

One of the things Michael really stressed was owning, and not being afraid of failure. Failure is inevitable; failure is still valid data. Knowing what doesn’t work can be just as valuable as knowing what does work. The obvious caveat is that you need to learn from your failures. This means doing post-mortems and root cause analysis for every failure. Shit happens; and if you can learn from it, you can prevent (or at least mitigate) it next time. Getting your team on board with the need to get smarter about these things will pay off very quickly. You need to make sure that the environment tolerates failure (obviously, to an extent) because, as he explains, experimentation matters. Experimentation leads to innovation, but sometimes experiments fail. If people are too afraid to experiment, innovation is going to come to a grinding halt.

I also found his point about collaboration and communication particularly powerful. DevOps is supposed to be all about Dev and Ops folks getting together and living happily ever after, right? No. Yes, DevOps is about collaboration between Dev and Ops—but it’s also about a culture of trust and communication. Collaboration and communication are both two-way streets. Michael insisted that Devs need to be trusted to do some things that Ops is normally uncomfortable with, but that they need to be held accountable. For example, for every upgrade, there needs to be a Dev on call to champion the upgrade. He also insisted that Devs be included in pager duty for production servers. This allows Dev to get a better idea of how their code affects the system (and I’m using that term literally). This all boils down to breaking down the silos.  Silos are bad because people lose perspective. Dev and Ops are working on the same system; the only way to truly understand the system is to take a systems approach. Taking a systems approach is central to the DevOps philosophy. But not everybody thinks that way; and that’s one of the hardest things to change in a culture. The systems approach is a must for DevOps to succeed in an organization; own it, sell it, and don’t stop until everybody is thinking about the system instead of their own corner of the system.

Michael’s presentation was followed by Leo Zhadanovsky’s presentation [“The Road to The White House with Puppet and AWS”](http://sched.co/16fsYPi). Leo worked for the DNC and ran the Obama campaign’s IT organization. This tickled my particularly bleeding heart, but regardless of your politics, the tech was incredibly cool. Head on over to [http://awsofa.info](http://awsofa.info) and take a look at the infrastructure—you won’t be disappointed. Of particular note was the colocation in US-West, which was done in response to the threat posed by Hurricane Sandy. They managed to colocate their entire infrastructure (in a reduced capacity) in just nine hours. NINE HOURS PEOPLE. It took three months to deploy the entire US-East environment, and NINE EFFING HOURS to deploy the entire US-West environment. Mind = blown.

Sam Bashton gave a presentation entitled [“Continuously Integrated Puppet in a Dynamic Environment”](http://sched.co/15azjY6) which turned out to be about something completely unrelated to continuous integration. I felt a little duped. He just talked about how they’ve implemented masterless Puppet in their environment. It wasn’t my cup of tea, but it was well done. I did enjoy his comparison of servers to cattle. He explained that most people treat servers like pets; they treat them very tenderly, care for them when they’re sick, and keep them around until they die. But servers should be treated like cattle—only kept around while they’re needed and when they’re sick or they aren’t needed anymore they should be taken out back and shot. I found this particularly amusing as someone who’s never set foot on a farm.

Kris Buytaert closed with a fantastic presentation about [“Monitoring in an Infrastructure as Code Age”](http://sched.co/11MpuTk). I love this term “Infrastructure as Code” and I promise to start incorporating it into everyday conversation. Kris was great to listen to. He boiled DevOps down to CLAMS which stood for culture, lean, automation, monitoring and metrics, and sharing. I’m going to get t-shirts made. Kris’s presentation can basically be summarized in four words: MONITOR ALL THE THINGS! He talked about making metrics central to the culture of your team (my coworker and I shared a frustrated glance at this). I would also recommend watching for this presentation on the PuppetLabs YouTube channel; I am not doing the presentation justice here.

I’ve attempted to boil down the top talking points presented at PuppetConf into some helpful aphorisms.

- **Infrastructure as code IS code, and it must be treated as such**. This means version control, code review, testing, and continuous integration. We’re all software developers now.
- **Stop writing shit modules**. Write modules that are modular (ZOMG! Modular modules!) and sharable.
- **Externalizing data from your modules from the beginning is good**. Do it from the start and avoid refactoring. Heira is your friend.
- **If you aren’t testing your modules, you’re doing it wrong**. There’s no excuse anymore. The tooling is there; use it.
- **Roles and profiles help keep your modules modular and flexible**. Roles and profiles allow you to create modules you can share. They are a good idea.
- **Follow the style guide**. We need to get more pythonic about the Puppet language. Use puppet-lint. Seriously, do it.
- **Puppet is a programming language**. Accept it folks. Puppet has become more and more robust and the sooner you realize it’s not just a DSL, the sooner you can start writing really robust modules. But Puppet also has a long way to go.
- **Remember the UNIX model**. Pick one thing and do it really really well.

I’ll close this post with some Python wisdom which is particularly applicable:

	>>> import this
	The Zen of Python, by Tim Peters
	
	Beautiful is better than ugly.
	Explicit is better than implicit.
	Simple is better than complex.
	Complex is better than complicated.
	Flat is better than nested.
	Sparse is better than dense.
	Readability counts.
	Special cases aren't special enough to break the rules.
	Although practicality beats purity.
	Errors should never pass silently.
	Unless explicitly silenced.
	In the face of ambiguity, refuse the temptation to guess.
	There should be one-- and preferably only one --obvious way to do it.
	Although that way may not be obvious at first unless you're Dutch.
	Now is better than never.
	Although never is often better than *right* now.
	If the implementation is hard to explain, it's a bad idea.
	If the implementation is easy to explain, it may be a good idea.
	Namespaces are one honking great idea -- let's do more of those!