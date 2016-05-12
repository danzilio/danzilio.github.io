---
layout: post
title: "Puppet Design Patterns: The Strategy Pattern"
date: 2016-05-12
tags:
  - blogmonth
  - 30in30
  - blog month 2016
  - Puppet
  - design patterns
comments: true
---

I'm currently in the process of putting together a talk about Design Patterns for Puppet, so I figured I'd blog a bit about it along the way. I'm pretty passionate about patterns because I think they really help you understand a problem as well as the dynamics of the language you're working in.

Design patterns are frequently used solutions to common problems. They tend to emerge naturally, and are usually observed rather than invented. The seminal work on design patterns was _Design Patterns: Elements of Reusable Object-Oriented Software_, commonly referred to as the Gang of Four (GoF) book. If you're interested in understanding more about design patterns in general, I highly recommend you pick up a copy of the GoF book as well as _Design Patterns in Ruby_ by Russ Olsen.

Not all of the GoF patterns can be directly applied to Puppet, and most of the patterns that do apply need a little bit of massaging to get there. This is mostly due to the fact that Puppet is not an object oriented programming language. That being said, I think there are definitely some lessons to be learned from the GoF when it comes to Puppet. There's also great value in simply identifying a pattern and giving it a name.

## The Strategy Pattern
The Strategy pattern is used when you have a part of an algorithm that must vary under certain conditions. The Strategy pattern uses **composition** to achieve that variation. The GoF describes this as "pull[ing] the algorithm out into a separate object."

There are two types of classes in the Strategy pattern: the **strategy** and the **context**. The **strategy** classes are parts of the code that need to vary; they are broken out into separate classes and (ideally) implement a common interface. The **context** class uses the strategy classes to achieve some complex behavior while abstracting the implementation from the user.

We often see the Strategy pattern used when trying to make our modules work across various platforms. Since Debian and RedHat based Linux distributions use different package managers, we must often vary our repository management logic to accommodate for the differences in configuration semantics and primitives available. Let's take a look at the `puppetlabs/rabbitmq` module for a real-world example of the Strategy pattern.

## Strategy in Action: `puppetlabs/rabbitmq`

In the [`rabbitmq`](https://github.com/puppetlabs/puppetlabs-rabbitmq/blob/5.4.0/manifests/init.pp) class there's a `manage_repos` parameter to enable or disable the management of the package repository. The `rabbitmq` module supports `yum` and `apt` based distributions and breaks the logic to configure those package managers in two separate classes: `rabbitmq::repo::rhel` and `rabbitmq::repo::apt`. These are the **strategy** classes. Let's take a look at how those classes are used in the `rabbitmq` class.

```puppet
  if $manage_repos != false {
    case $::osfamily {
      'RedHat', 'SUSE': {
          include '::rabbitmq::repo::rhel'
          $package_require = undef
      }
      'Debian': {
        class { '::rabbitmq::repo::apt' :
          key_source  => $package_gpg_key,
          key_content => $key_content,
        }
        $package_require = Class['apt::update']
      }
      default: {
        $package_require = undef
      }
    }
  } else {
    $package_require = undef
  }
```

As you can see, this code looks at the `osfamily` fact to determine which `rabbitmq::repo` class to include. For `RedHat` and `SUSE` based distrbutions, the `rabbitmq` class includes the `rabbitmq::repo::rhel` class. For `Debian` based distributions, it includes the `rabbitmq::repo::apt` class. The `rabbitmq` class is the user of the strategy classes; the GoF called this the **context** class.

The Strategy pattern here allows us to abstract away the innards of repository management based on the `osfamily` running on the client. By breaking out the repository management into separate classes, we've made the code more modular and easier to read. We've achieved a better separation of concerns by delegating the repository management to a separate class. We've also made it easier and less risky to add new platforms to this module.
