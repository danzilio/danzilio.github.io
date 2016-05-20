---
layout: post
title: "Puppet Design Patterns: The Factory Pattern"
date: 2016-05-20
tags:
  - blogmonth
  - 30in30
  - blog month 2016
  - Puppet
  - design patterns
comments: true
---

Yesterday, at [PuppetCamp Boston](https://puppet.com/community/events/camp/puppet-camp-boston-2016),
I gave a talk on Puppet Design Patterns. Last week, I wrote a blog post about
the [Strategy Pattern](http://blog.danzil.io/2016/05/12/puppet-design-patterns-strategy.html).
In this post I'll talk about the Factory Pattern.

## What's in a name?
You may be familiar with the Factory Pattern by a different name: the
`create_resources` pattern. I've decided to call this the Factory Pattern
because the `create_resources` function is no longer the only method available
to implement this pattern. With the new language features introduced in Puppet
4, we can now use iteration natively in the DSL instead of relying on the
`create_resources` function.

I chose the name "Factory" because this pattern closely aligns with the GoF
Factory Method and Abstract Factory patterns. Others have used the Factory
Pattern terminology to refer generically to an object that instantiates other
objects based on some input or message. While Puppet doesn't have objects (in
the traditional sense), it does have resources. A Puppet class can act as a
resource factory, creating other resources based on input from its interface.
This is becoming an increasingly common pattern.

## The Factory Pattern in Action
The Factory Pattern allows you to have a single point of entry for your module.
Your entry point can use the information passed to it to determine what
resources to create. Let's take a look at an example using the
`create_resources` function.

```puppet
class filefactory (
  $files = {}
) {
  validate_hash($files)

  unless empty($files) {
    create_resources('file', $files)
  }
}
```

The example above is contrived. The `filefactory` class does nothing but create
other resources. In reality, your module would likely have other
responsibilities in addition to creating resources. Here, we accept a single
parameter named `$files` and we expect the data passed to `$files` to be a hash.
We check to make sure the `$files` variable is not empty, and then we pass that
hash to the `create_resources` function where we tell it to create the resource
type `file`. Let's take a look at what this would look like in Puppet 4.

```puppet
class filefactory (
  Hash $files
) {
  $files.each |$name, $resource| {
    file { $name:
      * => $resource
    }
  }
}
```

In the Puppet 4 example, we can use iteration inside the DSL instead of
delegating that to the `create_resources` function. In this example we still
accept a `$files` parameter, but we don't provide a default here. We also rely
on the type system to ensure that the data passed to `$files` is a hash instead
of using the `validate_hash` function. We iterate over the `$files` hash and
unpack the key as `$name` and the value as `$resource`. We then create a file,
passing `$name` as the resource title, and we use the splat operator to pass the
`$resource` hash. The splat operator passes hash keys to the resource as
parameter names, and the hash values as the parameter values.

In the above examples, the `filefactory` class is the resource factory. It
creates new resources based on the data passed to the factory's interface. Let's
take a look at a real-world example of this.

## The Factory Pattern in `ghoneycutt/nrpe`
The Factory Pattern allows you to create resources inside your entry point. We
see this in a number of modules, but let's take a look at Garrett Honeycutt's
`nrpe` module [here](https://github.com/ghoneycutt/puppet-module-nrpe). This
module consists of one class and one defined type. The class installs and
manages the `nrpe` service while the defined type allows the user to configure
`nrpe` plugins and checks. Garrett uses the Factory Pattern to allow his users
to simply `include nrpe` and let the `nrpe` class create the `nrpe::plugin`
resources based on Hiera data. Let's take a look at the code (I've truncated the
code to focus on the parts we care about).

```puppet
class nrpe (
  ...

  $plugins                          = undef,
  $hiera_merge_plugins              = false,

  ...

) {

  ...

  if is_string($hiera_merge_plugins) {
    $hiera_merge_plugins_bool = str2bool($hiera_merge_plugins)
  } else {
    $hiera_merge_plugins_bool = $hiera_merge_plugins
  }

  ...

  if $plugins != undef {
    if $hiera_merge_plugins_bool {
      $plugins_real = hiera_hash(nrpe::plugins)
    } else {
      $plugins_real = $plugins
    }
    validate_hash($plugins_real)
    create_resources('nrpe::plugin',$plugins_real)
  }
}
```

Here, as you can see, Garrett's `nrpe` class accepts the `$plugins` and
`$hiera_merge_plugins` parameters. If the `$plugins` parameter is not `undef`,
it passes that data to the `create_resources` function depending on whether the
user has enabled hash merging via the `$hiera_merge_plugins` parameter. This
use of the Factory Pattern provides a safe and simple interface to this module.

## A Note on `create_resources`
Many people in the community consider use of the `create_resources` function as
an antipattern. This is mainly due to the fact that it was created as a
workaround for the lack of iteration support in older versions of Puppet. In the
past, the `create_resources` function was often misused. I personally think that
`create_resources` is perfectly fine when used judiciously. Now that we have
powerful iteration support in Puppet 4, hopefully the Factory Pattern can begin
to be seen at as a more 'legitimate' pattern without the baggage of
`create_resources` weighing it down.

## A Note on the Name
I don't claim to be an authority on names. I think this pattern closely
resembles what others refer to as a Factory. Hopefully this terminology will
catch on, but I don't represent the community as a whole. This post is meant to
start the discussion :)
