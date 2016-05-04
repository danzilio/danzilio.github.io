---
layout: post
title: "RSpec For Ops Part 2: Diving in with rspec-puppet"
date: 2016-05-03
tags:
  - RSpec
  - RSpec for Ops
  - TDD
  - Testing
  - blogmonth
  - 30in30
  - blog month 2016
comments: true
---

This is the second part in a series of blog posts on RSpec for Ops. See the first post [here](http://blog.danzil.io/2016/02/05/rspec-for-ops-part-1-essentials.html).

Now that we've got a good understanding of some of the RSpec primitives, we'll dive into how these can be used when testing infrastructure. I'm going to use `rspec-puppet` for my examples here. In a forthcoming post I'll use `serverspec` for a more agnostic approach. If you're not a Puppet user, don't worry, the two main Infracode testing tools (Beaker and Kitchen) both make heavy use of Serverspec.

There are two general types of tests that you'll write as an Infracoder: **acceptance** and **unit** tests. We'll use `rspec-puppet` for **unit** tests.

## Unit tests

Unit tests are written to test a specific bit of code. With `rspec-puppet` we test our Puppet *classes*, *defined types*, and *resources*. We'll use `rspec-puppet` to define a specification for a Puppet class. This class will install a package, place a configuration file, and start a service. Because we're using test **driven** development, we'll write our tests before we write the code. Let's get started:

```ruby
require 'spec_helper'

describe 'ntp' do
  it { is_expected.to compile }
end
```

Here we have written a test for the `ntp` class. We have a single example with the expectation that the catalog will compile with the `ntp` class included. There's some `rspec-puppet` magic going on here, so let's delve into this. First, you'll notice that I'm not using an explicit subject here, we're relying on the implicit subject using the first argument passed to the `describe` method. The `rspec-puppet` helpers assume the first argument to the `describe` method is the name of a Puppet class. It will automatically generate a manifest with `include ntp` as the content and will use that manifest to compile a catalog. The resulting catalog object then gets stored as the `subject` for use in our examples. This test has a single expectation. Using the `compile` matcher, we set the expectation that the `catalog` will successfully compile. When we run this, we'll get an error:

```
ntp
  should compile into a catalogue without dependency cycles (FAILED - 1)

Failures:

  1) ntp should compile into a catalogue without dependency cycles
     Failure/Error: it { is_expected.to compile }
       error during compilation: Evaluation Error: Error while evaluating a Function Call, Could not find class ::ntp for mbp.danzil.io at line 1:1 on node mbp.danzil.io
     # ./spec/classes/ntp_spec.rb:4:in `block (2 levels) in <top (required)>'

Finished in 0.17856 seconds (files took 0.69616 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/classes/ntp_spec.rb:4 # ntp should compile into a catalogue without dependency cycles
```

This error is expected, since we never actually wrote the code to implement the `ntp` class. Let's do that now:

```puppet
class ntp { }
```

The goal is to write the *minimum* amount of code to make the tests pass. All our tests are looking for is that the `ntp` class exists and that it compiles. The code above is all we need to meet that requirement. If we run the tests again, we'll see:

```
ntp
  should compile into a catalogue without dependency cycles

Finished in 0.18003 seconds (files took 0.69723 seconds to load)
1 example, 0 failures
```

Now, this class doesn't do anything, so it's not particularly useful. Let's install the `ntp` package:

```ruby
require 'spec_helper'

describe 'ntp' do
  it { is_expected.to compile }
  it { is_expected.to contain_package('ntpd') }
end
```

We've added a new example to our `ntp` group with the expectation that the catalog will contain the `ntpd` package. We're using the `contain_package` matcher to assert this expectation. When we run our new tests we'll have a new failure:

```
ntp
  should compile into a catalogue without dependency cycles
  should contain Package[ntpd] (FAILED - 1)

Failures:

  1) ntp should contain Package[ntpd]
     Failure/Error: it { is_expected.to contain_package('ntpd') }
       expected that the catalogue would contain Package[ntpd]
     # ./spec/classes/ntp_spec.rb:5:in `block (2 levels) in <top (required)>'

Finished in 1.47 seconds (files took 0.79995 seconds to load)
2 examples, 1 failure

Failed examples:

rspec ./spec/classes/ntp_spec.rb:5 # ntp should contain Package[ntpd]
```

The output is complaining that the catalog is missing the expected `ntpd` package. Let's add the code to make that test pass:

```puppet
class ntp {
  package { 'ntpd': }
}
```

Now our tests should pass:

```
ntp
  should compile into a catalogue without dependency cycles
  should contain Package[ntpd]

Finished in 1.41 seconds (files took 0.77791 seconds to load)
2 examples, 0 failures
```

You'll continue with this until you have a functioning module. I'm going to skip ahead a bit so we can move on to more interesting things. Here's the rest of the spec I'm going to write for this module:

```ruby
require 'spec_helper'

describe 'ntp' do
  it { is_expected.to compile }
  it { is_expected.to contain_package('ntpd') }

  it 'should contain the ntp configuration file' do
    is_expected.to contain_file('/etc/ntpd.conf')
    is_expected.to contain_file('/etc/ntpd.conf').that_requires('Package[ntpd]')
    is_expected.to contain_file('/etc/ntpd.conf').that_notifies('Service[ntpd]')
  end

  it { is_expected.to contain_service('ntpd') }
end
```

When I run this test, we'll see the expected failures:

```
ntp
  should compile into a catalogue without dependency cycles
  should contain Package[ntpd]
  should contain the ntp configuration file (FAILED - 1)
  should contain Service[ntpd] (FAILED - 2)

Failures:

  1) ntp should contain the ntp configuration file
     Failure/Error: is_expected.to contain_file('/etc/ntpd.conf')
       expected that the catalogue would contain File[/etc/ntpd.conf]
     # ./spec/classes/ntp_spec.rb:8:in `block (2 levels) in <top (required)>'

  2) ntp should contain Service[ntpd]
     Failure/Error: it { is_expected.to contain_service('ntpd') }
       expected that the catalogue would contain Service[ntpd]
     # ./spec/classes/ntp_spec.rb:13:in `block (2 levels) in <top (required)>'

Finished in 1.39 seconds (files took 0.66577 seconds to load)
4 examples, 2 failures

Failed examples:

rspec ./spec/classes/ntp_spec.rb:7 # ntp should contain the ntp configuration file
rspec ./spec/classes/ntp_spec.rb:13 # ntp should contain Service[ntpd]
```

I'll implement those features in the code:

```puppet
class ntp {
  package { 'ntpd': }
  file { '/etc/ntpd.conf':
    ensure  => file,
    content => file('ntp/ntpd.conf'),
    require => Package['ntpd'],
    notify  => Service['ntpd'],
  }
  service { 'ntpd':
    ensure => running,
    enable => true,
  }
}
```

When I run my tests, all of my examples pass:

```
ntp
  should compile into a catalogue without dependency cycles
  should contain Package[ntpd]
  should contain the ntp configuration file
  should contain Service[ntpd]

Finished in 2.03 seconds (files took 0.73826 seconds to load)
4 examples, 0 failures
```

Great! We have a functioning Puppet module, but not a particularly useful one. This is an admittedly contrived example. It only works on platforms that have a package named `ntpd` and call that service `ntpd` and look for a configuration file at `/etc/ntpd.conf`. Let's make this code a little more useful by adding some parameters:

```ruby
require 'spec_helper'

describe 'ntp' do
  let(:params) do { {
    :package_name => 'ntpd',
    :config_file  => '/etc/ntpd.conf',
    :service_name => 'ntpd'
  } }

  it { is_expected.to compile }
  it { is_expected.to contain_package('ntpd') }

  it 'should contain the ntp configuration file' do
    is_expected.to contain_file('/etc/ntpd.conf')
    is_expected.to contain_file('/etc/ntpd.conf').that_requires('Package[ntpd]')
    is_expected.to contain_file('/etc/ntpd.conf').that_notifies('Service[ntpd]')
  end

  it { is_expected.to contain_service('ntpd') }
end
```

The test above uses the `let` method to declare `params`. `rspec-puppet` looks at `params` for a hash of parameters to pass to the Puppet class. Let's run this test:

```
ntp
  should compile into a catalogue without dependency cycles (FAILED - 1)
  should contain Package[ntpd] (FAILED - 2)
  should contain the ntp configuration file (FAILED - 3)
  should contain Service[ntpd] (FAILED - 4)

Failures:

  1) ntp should compile into a catalogue without dependency cycles
     Failure/Error: it { is_expected.to compile }

       error during compilation: Evaluation Error: Error while evaluating a Resource Statement, Class[Ntp]:
         has no parameter named 'package_name'
         has no parameter named 'config_file'
         has no parameter named 'service_name' at line 1:1 on node mbp.danzil.io
     # ./spec/classes/ntp_spec.rb:10:in `block (2 levels) in <top (required)>'

  2) ntp should contain Package[ntpd]
     Failure/Error: it { is_expected.to contain_package('ntpd') }

     Puppet::PreformattedError:
       Evaluation Error: Error while evaluating a Resource Statement, Class[Ntp]:
         has no parameter named 'package_name'
         has no parameter named 'config_file'
         has no parameter named 'service_name' at line 1:1 on node mbp.danzil.io

  3) ntp should contain the ntp configuration file
     Failure/Error: is_expected.to contain_file('/etc/ntpd.conf')

     Puppet::PreformattedError:
       Evaluation Error: Error while evaluating a Resource Statement, Class[Ntp]:
         has no parameter named 'package_name'
         has no parameter named 'config_file'
         has no parameter named 'service_name' at line 1:1 on node mbp.danzil.io

  4) ntp should contain Service[ntpd]
     Failure/Error: it { is_expected.to contain_service('ntpd') }

     Puppet::PreformattedError:
       Evaluation Error: Error while evaluating a Resource Statement, Class[Ntp]:
         has no parameter named 'package_name'
         has no parameter named 'config_file'
         has no parameter named 'service_name' at line 1:1 on node mbp.danzil.io

Finished in 0.18966 seconds (files took 0.69537 seconds to load)
4 examples, 4 failures

Failed examples:

rspec ./spec/classes/ntp_spec.rb:10 # ntp should compile into a catalogue without dependency cycles
rspec ./spec/classes/ntp_spec.rb:11 # ntp should contain Package[ntpd]
rspec ./spec/classes/ntp_spec.rb:13 # ntp should contain the ntp configuration file
rspec ./spec/classes/ntp_spec.rb:19 # ntp should contain Service[ntpd]
```

I've truncated the output here for clarity. You'll see that everything blew up because I'm trying to pass parameters to a class that doesn't accept parameters! Let's add those parameters to my Puppet class:

```puppet
class ntp (
  $package_name = 'ntpd',
  $config_file  = '/etc/ntpd.conf',
  $service_name = 'ntpd',
){
  package { $package_name: }
  file { $config_file:
    ensure  => file,
    content => file('ntp/ntpd.conf'),
    require => Package[$package_name],
    notify  => Service[$service_name],
  }
  service { $service_name:
    ensure => running,
    enable => true,
  }
}
```

Now when we run our tests, we'll see a much greener (and more familiar) output:

```
ntp
  should compile into a catalogue without dependency cycles
  should contain Package[ntpd]
  should contain the ntp configuration file
  should contain Service[ntpd]

Finished in 1.76 seconds (files took 0.66553 seconds to load)
4 examples, 0 failures
```

### Interfaces are Key

We've just refactored this code to accept parameters. The most important testing practice (IMHO) is to **focus on the interface, not the implementation**. By adding parameters to this class, we've created an interface. Our parameters are our module's API. Well written tests should validate our interfaces and ensure that the data we're passing to our interface is making it to our resources the way we expect it to. When we focus on the implementation we end up just rewriting our code in another language, focusing on the interface allows us to focus on the bits of our code that we *really* care about.

Now that we have some parameters, we can start testing different contexts. Contexts are helpful when testing your module's interface under different conditions. We'll want to validate our interface in a number of different ways. I'm going to restructure our tests to focus on the interface:

```ruby
require 'spec_helper'

describe 'ntp' do
  context 'with the default parameters' do
    it { is_expected.to compile }
    it { is_expected.to contain_package('ntpd') }

    it 'should contain the ntp configuration file' do
      is_expected.to contain_file('/etc/ntpd.conf')
      is_expected.to contain_file('/etc/ntpd.conf').that_requires('Package[ntpd]')
      is_expected.to contain_file('/etc/ntpd.conf').that_notifies('Service[ntpd]')
    end

    it { is_expected.to contain_service('ntpd') }
  end

  context 'when specifying the package_name parameter' do
    let(:params) { { :package_name => 'ntp' } }
    it { is_expected.to contain_package('ntp') }
    it { is_expected.to contain_file('/etc/ntpd.conf').that_requires('Package[ntp]') }
  end

  context 'when specifying the config_file parameter' do
    let(:params) { { :config_file => '/etc/ntp.conf' } }
    it { is_expected.to contain_file('/etc/ntp.conf') }
  end

  context 'when specifying the service_name parameter' do
    let(:params) { { :service_name => 'ntp' } }
    it { is_expected.to contain_service('ntp') }
    it { is_expected.to contain_file('/etc/ntpd.conf').that_notifies('Service[ntp]') }
  end
end
```

As you can see, I've added contexts for each of my individual parameters, as well as a context that covers the default parameter values. I am testing to make sure that the parameter values are being consumed correctly in my code. These tests will already pass since we're not implementing any new features here just improving the tests.

```
ntp
  with the default parameters
    should compile into a catalogue without dependency cycles
    should contain Package[ntpd]
    should contain the ntp configuration file
    should contain Service[ntpd]
  when specifying the package_name parameter
    should contain Package[ntp]
    should contain File[/etc/ntpd.conf] that requires Package[ntp]
  when specifying the config_file parameter
    should contain File[/etc/ntp.conf]
  when specifying the service_name parameter
    should contain Service[ntp]
    should contain File[/etc/ntpd.conf] that notifies Service[ntp]

Finished in 2.11 seconds (files took 0.78148 seconds to load)
9 examples, 0 failures
```

Remember, the goal is not to rewrite your Puppet code in RSpec, it's to ensure that your code is behaving the way you expect it to.

### Stay Tuned

In the next post in this series, I'll dive a little deeper into module development and `rspec-puppet` best practices. I'll also follow up with a post where I'll use `serverspec` to write some **acceptance** tests for this example code.
