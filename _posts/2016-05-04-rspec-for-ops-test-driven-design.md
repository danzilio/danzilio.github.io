---
layout: post
title: "RSpec For Ops Part 3: Test driven design with rspec-puppet"
date: 2016-05-04
tags:
  - RSpec
  - RSpec for Ops
  - TDD
  - Testing
comments: true
published: false
---

# RSpec for Ops Part 3: Test driven design with `rspec-puppet`
This is the third part in a series of blog posts on RSpec for Ops. See the first
post
[here](http://blog.danzil.io/2016/02/05/rspec-for-ops-part-1-essentials.html)
and the second post
[here](http://blog.danzil.io/2016/05/03/rspec-for-ops-rspec-puppet.html).

## Designing for Testability
Once you get more comfortable with TDD for Puppet, you'll start to realize that
there's a particular way to design your code to make it easier to test. I often
hear complaints from newcomers to TDD to the effect of "what's the point of TDD
if I have to change how I write my code just for these tests?" This is a very
good question. To answer this, let's take a look at what makes code easier to
test.

### Interfaces
As I mentioned in my last post, the key to writing good tests is to focus on
your module's **interfaces**. Your module should have a well defined interface.
When I say _interface_, I'm talking about an Application Programming Interface
(or API) which, in Puppet, is created using **parameters**. Your goal should be
to validate your module's interface, ensuring that all **branch conditions** are
covered by your tests. It's much easier to test code that has an interface
rather than trying to test conditions that are hardcoded inside the module.
Let's look at an example:

```puppet
class ssh {
  $package_name = $osfamily ? {
    'gentoo' => 'openssh',
    default  => 'openssh-server'
  }

  package { $package_name:
    ensure => installed
  }
}
```

The example above installs an ssh server. We want to support more than one,
platform so we've used a selector to determine the package name based on the
`osfamily` fact. Let's see what the test looks like for this:

```ruby
describe 'ssh' do
  context 'on gentoo' do
    let(:facts) {{ :osfamily => 'gentoo' }}
    it { is_expected.to contain_package 'openssh' }
  end

  context 'on redhat' do
    let(:facts) {{ :osfamily => 'redhat' }}
    it { is_expected.to contain_package 'openssh-server' }
  end
end
```

This test looks simple enough, right? It's testing all of our branch conditions
and gets pretty good coverage. What happens when you want to add a new platform?
What happens when the package name changes? You'll have to write new tests to
ensure that your branch conditions are covered. Let's see what this looks like
with an interface:

```puppet
class ssh (
  $package_name = $ssh::params::package_name,
) inherits ssh::params {

  package { $package_name:
    ensure => installed
  }
}
```

This example uses the `$package_name` parameter to set the package name and gets
its default value from the `params` class. Let's see what the test would look
like for this class:

```ruby
describe 'ssh' do
  let(:params) {{ :package_name => 'openssh-server' }}
  it { is_expected.to contain_package 'openssh-server' }
end
```

This test validates our interface and makes sure that the value we set at
`$package_name` is passed to the package resource as we expected. If some part
of the code changes the way this parameter is passed to the package resource,
the tests will fail. To be sure, you should also validate your branching logic
from your `params` class, but focusing on the interface here has forced us to
redesign our code in a way that has some nice corollary benefits. This class is
no longer dependent on embedded logic, so it is more resilient to unforeseen use
cases. Our tests needs to know even less about our implementation code, which
makes them more resilient and easier to maintain.

### Composition and Single Responsibility
Composition is a practice where you combine discrete bits of code to achieve a
desired behavior. Using composition along with classes that have a single
responsibility, you can achieve complex behaviors that are easy to test, read,
and extend. To illustrate, let's take a look at a bit of code with many
responsibilities.

```puppet
class ssh {
  package { 'openssh-server':
    ensure => installed,
  }
  file { '/etc/ssh/sshd_config':
    ensure  => file,
    content => template('ssh/sshd_config.erb'),
    require => Package['openssh-server'],
    notify  => Service['sshd'],
  }
  service { 'sshd':
    ensure  => running,
    enabled => true,
  }
}
```

This example does three things: it installs a package, places a configuration
file, and ensures a service. This class has three separate responsibilities.
This means it has three behaviors that we'd need to test. Let's look at the
tests for this module.

```ruby
describe 'ssh' do
  it { is_expected.to contain_package 'openssh-server' }

  it 'should configure ssh' do
    is_expected.to contain_file('/etc/ssh/sshd_config').that_requires('Package[openssh-server]')
    is_expected.to contain_file('/etc/ssh/sshd_config').that_notifies('Service[sshd]')
  end

  it 'should start sshd' do
    is_expected.to contain_service('sshd').with { :ensure => 'running', :enabled => true }
  end
end
```

This is not such a burden right now, but once this module gets big enough, it'll
be much easier to test these behaviors individually rather than together. In
fact, there's an established pattern to handle this exact example: the
**package, file, service** pattern. Let's refactor this module to use the
package, file, service pattern and see how the tests look.

```puppet
class ssh {
  include ssh::install
  include ssh::config
  include ssh::service

  Class['ssh::install'] ->
  Class['ssh::config'] ~>
  Class['ssh::service']
}

class ssh::install {
  package { 'openssh-server': ensure => installed }
}

class ssh::config {
  file { '/etc/ssh/sshd_config':
    ensure  => file,
    content => template('ssh/sshd_config.erb'),
  }
}

class ssh::service {
  service { 'sshd': ensure => running, enabled => true }
}
```

Each of these classes has a singular responsibility. The `ssh` class is
responsible for including the subclasses that make up the desired behavior for
the module, and ensuring that the resource ordering is correct. The
`ssh::install` class is responsible for installing the 'openssh-server' package.
The `ssh::config` class is responsible for configuring the `sshd` service.
Finally, the `ssh::service` class is responsible for managing the `sshd`
service.

```ruby
describe 'ssh' do
  it { is_expected.to contain_class('ssh::install').that_comes_before('Class[ssh::config]') }
  it { is_expected.to contain_class('ssh::config').that_notifies('Class[ssh::service]') }
  it { is_expected.to contain_class('ssh::service') }
end

describe 'ssh::install' do
  it { is_expected.to contain_package('openssh-server') }
end

describe 'ssh::config' do
  it { is_expected.to contain_file('/etc/ssh/sshd_config') }
end

describe 'ssh::service' do
  is_expected.to contain_service('sshd').with({ :ensure => 'running', :enabled => true })
end
```

Writing classes with a single responsibility makes writing tests easier because
you can focus on a single behavior at a time. You can group your tests based on
the behaviors they're testing, and you can test different conditions that may
affect your code's behavior discretely. Once you've refactored this code into
discrete classes with a single responsibility, you can achieve the same overall
behavior with composition.

## Designing for Testability is Good Overall Design
Design for testability just so happens to be good overall design. These design
principles include **programming to an interface**, **composition**, and
**single responsibility**. If you encounter a class that's hard to test, think
about why it's hard. Chances are you've identified a good opportunity to
redesign your code because test driven development fleshes out hidden design
flaws. At first it may seem like you're massaging your code to fit your testing
strategy but, in reality, you're fixing design flaws that make it harder to
read, maintain, and change your code.

Tests should be designed to be largely independent of your code's internals. If
you find yourself battling fragile tests, you've probably coupled your tests to
your code too tightly. Take a look at your test code, as well as your
implementation code, and try and identify missing or poorly defined interfaces.
Think about the assumptions you're making in your code. Make sure you're not
hard coding those assumptions into your tests to make up for missing interfaces.

TDD may seem counterintuitive at first, but that will quickly pass. The benefits
of TDD are proven, and become apparent rather quickly. After a while you'll
notice yourself writing code that's simpler, more modular, more readily able to
handle unforeseen use cases, and easier to maintain.
