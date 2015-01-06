---
layout: post
title: "Puppet 4 Preparedness: Testing modules with the future parser and repec-puppet"
date: 2015-01-05
tags:
  - future parser
  - Puppet
  - Puppet 4
  - rspec-puppet
comments: true
redirect_from: "/2015/01/06/puppet-4-preparedness-testing-modules-with-the-future-parser-and-rspec-puppet/"
---

If you’re like me, you’re interested in making the transition to Puppet 4 as quickly and seamlessly as possible (er...at least a little easier and faster than from 2.7 to 3). Now that the Puppet 4 language is [nearly complete](https://docs.puppetlabs.com/puppet/3.7/reference/release_notes.html#feature-nearly-final-implementation-of-the-puppet-4-language), we can use the future parser in Puppet 3.7 to test our code to make sure it’s ready for the switch. Thanks to some recent changes in rspec-puppet, we can more easily integrate testing with the future parser into our CI workflow. I recently added this functionality to my [virtualbox](https://forge.puppetlabs.com/danzilio/virtualbox) module on the Forge. Head on over to [GitHub](https://github.com/danzilio/danzilio-virtualbox) if you’re interested in the full example.

First, you’ll need to use rspec-puppet version 2.0.0 or higher. As of this writing, version 2.0.0 has been tagged in the GitHub repo but has yet to be uploaded to rubygems.org. You’ll probably need to pull the code directly from GitHub by adding the `:git` and `:ref` parameters to the `rsepc-puppet` entry in your `Gemfile`:

```ruby
gem 'rspec-puppet',
  :git => 'https://github.com/rodjek/rspec-puppet.git',
  :ref => 'v2.0.0'
```

Next, you’ll need to tweak your `spec_helper` a little bit to enable the future parser:

```ruby
# spec/spec_helper.rb
require 'puppetlabs_spec_helper/module_spec_helper'

if ENV['PARSER'] == 'future'
  RSpec.configure do |c|
    c.parser = 'future'
  end
end
```

The block of code I’ve added to `spec_helper.rb` checks the `PARSER` environment variable and enables the future parser based on the value of that environment variable. Now if you want to run your `rspec-puppet` tests with the future parser, you just need to set the `PARSER` environment variable to `future`. You can do this at the command line like this:

	$ PARSER='future' bundle exec rake test

You can also do this very easily with Travis CI using the `.travis.yml` file:

```ruby
# .travis.yml
---
language: ruby
bundler_args: --without development
before_install: rm Gemfile.lock || true
sudo: false
rvm:
  - 1.8.7
  - 1.9.3
  - 2.0.0
  - 2.1.0
script: bundle exec rake test
env:
  - PUPPET_VERSION="~> 2.7.0"
  - PUPPET_VERSION="~> 3.1.0"
  - PUPPET_VERSION="~> 3.2.0"
  - PUPPET_VERSION="~> 3.3.0"
  - PUPPET_VERSION="~> 3.4.0"
  - PUPPET_VERSION="~> 3.5.0"
  - PUPPET_VERSION="~> 3.6.0"
  - PUPPET_VERSION="~> 3.7.0"
matrix:
  exclude:
  - rvm: 1.9.3
    env: PUPPET_VERSION="~> 2.7.0"
  - rvm: 2.0.0
    env: PUPPET_VERSION="~> 2.7.0"
  - rvm: 2.0.0
    env: PUPPET_VERSION="~> 3.1.0"
  - rvm: 2.1.0
    env: PUPPET_VERSION="~> 2.7.0"
  - rvm: 2.1.0
    env: PUPPET_VERSION="~> 3.1.0"
  - rvm: 2.1.0
    env: PUPPET_VERSION="~> 3.2.0"
  - rvm: 2.1.0
    env: PUPPET_VERSION="~> 3.3.0"
  - rvm: 2.1.0
    env: PUPPET_VERSION="~> 3.4.0"
  include:
  - rvm: 1.8.7
    env:
    - PUPPET_VERSION="~> 3.7.0"
    - PARSER="future"
  - rvm: 1.9.3
    env:
    - PUPPET_VERSION="~> 3.7.0"
    - PARSER="future"
  - rvm: 2.0.0
    env:
    - PUPPET_VERSION="~> 3.7.0"
    - PARSER="future"
  - rvm: 2.1.0
    env:
    - PUPPET_VERSION="~> 3.7.0"
    - PARSER="future"
```

The easiest way I found, based on my test matrix, was using the `include` directive in `.travis.yml` but if you can think of a better way, let me know! In fact, if there’s a better way to do any of this, please let me know!
