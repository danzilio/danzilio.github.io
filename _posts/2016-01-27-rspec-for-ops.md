---
layout: post
title: "RSpec For Ops"
date: 2016-01-27
tags:
  - RSpec
  - TDD
  - Testing
comments: true
published: false
---

# RSpec for Ops

I've noticed a lot of interest in RSpec from ops folks lately, likely driven by infracode testing tools like [Kitchen](http://kitchen-ci.org), [Beaker](https://github.com/puppetlabs/beaker), and [Serverspec](http://serverspec.org). I've also noticed a lot of ops folks using `rspec` without really understanding some of the fundamentals. This results in some suboptimal `rspec` code, and generally lots of headaches for newcomers to `rspec`. This blog post is an attempt to outline some of the basics of test driven development with `rspec` from the perspective of an Op :) I'll start by outlining `rspec` basics followed by some (hopefully) relatable examples.

# RSpec Basics

RSpec is a [behavior driven development](https://en.wikipedia.org/wiki/Behavior-driven_development) (BDD) framework for Ruby. BDD focuses on the behavior and expected outcomes of your code. RSpec provides a simple [domain-specific language](https://en.wikipedia.org/wiki/Domain-specific_language) that employs natural-lanaguage constructs for describing tests. As a result, BDD code is often very easy to read with inputs and expectations being very easy to identify. RSpec offers several primitives to model the behavior of your code including **groups**, **examples**, **contexts**, **expectations**, and **matchers**.

## Examples and Groups

One of the most fundamental primitives in `rspec` is the **example**. An example is the description of a behavior you expect your code to exhibit. Examples are declared using the `it` method and are often grouped together logically inside an (very creatively named) **example group**. Examples are comprised of a **subject** and an **expectation**. Example groups are declared with the `describe` method. Example groups make up your application's specification; they often end up looking a lot like lists of acceptance criteria. Here's a simple example:

```ruby
it { is_expected.to be true }
```

This code can be read as "it is expected to be true" and literally means that the **subject** of the test (we'll go into more detail in a minute) should return `true`. I didn't set a `subject` in the example above, but we can do that with the `let` method:

```ruby
let(:subject) { true }

it { is_expected.to be true }
```

The `let` method creates a [memoized](https://en.wikipedia.org/wiki/Memoization) helper method set to the value passed in the block. A memoized helper method is basically an `attr_reader` on an instance variable. Here's what the above `let` method effectively does under the hood:

```ruby
def subject
  @subject ||= true
end
```

The `let` method creates a new method based on the name you passed it as an argument. The code passed to the `let` method's block is then lazily evaluated and cached in the instance variable `@subject`. **Lazy evaluation** means that the code associated with this method is not evaluated until it is called the first time. This is in contrast to **eager evaluation** where the code is evaluated as soon as it is bound to a variable. This allows you to set some state that persists inside an example group, this also means that you can only use the `let` method inside an example group. Let's fix our example code:

```ruby
describe 'Truthiness' do
  let(:subject) { true }

  it { is_expected.to be true }
end
```

Here we have an example group named `Truthiness`, we've set the `subject` to `true`, and we've created an example with the expectation that our subject is `true`. Let's run this code and see what the output is:

```
Truthiness
  should equal true

Finished in 0.00107 seconds (files took 0.08412 seconds to load)
1 example, 0 failures
```

## Expectations and Matchers

An example is just a list of behaviors expected of our subject. Our example has one **expectation**: that the subject should be `true`. Our example above could also be expressed using the `expect` method:

```ruby
describe 'Truthiness' do
  let(:subject) { true }

  it { expect(subject).to be true }
end
```

This doesn't read as smoothly as it did with the `is_expected` method, which is just `expect` with an implicit `subject`. We can make this example more readable by passing a descriptive title to the `it` method:

```ruby
describe 'Truthiness' do
  let(:subject) { true }

  it 'should behave truthy' do
    expect(subject).to be true
  end
end
```

Now this example is easier to read. We can see that our subject `should behave truthy` and we have a list of expectations that need to be met for our example to pass. Let's add an expectation that our subject is not `false`:

```ruby
describe 'Truthiness' do
  let(:subject) { true }

  it 'should behave truthy' do
    expect(subject).to be true
    expect(subject).not_to be false
  end
end
```

These code examples are available on my GitHub [here](https://github.com/danzilio/rspec_for_ops). The example above can be found in [spec/truthiness_spec.rb](https://github.com/danzilio/rspec_for_ops/blob/master/spec/blog_examples/truthiness_spec.rb)

As you can see, examples can contain any number of expectations. You should use as many expectations as necessary to validate the behavior you're testing in an example. One element of an expectation that we haven't yet discussed is the **matcher**, which is really the heart of an expectation. Our example above only uses the `be` matcher but there are many others. Matchers describe the behavior you expect your subject to exhibit. In our example, we are using the `be` matcher to compare our subject's identity with our matcher.

There are lots of [built-in matchers](https://www.relishapp.com/rspec/rspec-expectations/v/3-4/docs/built-in-matchers); common matchers include `be`, `eq`, `exist`, `include`, `match`, `start_with`, and `respond_to`. RSpec has a robust matcher API that makes it very easy to write custom matchers. Infracode frameworks built on RSpec like Serverspec include a number of domain-specific matchers. Let's take a look at some different kinds of matchers:

```ruby
describe 'Hello world!' do
  it 'should say hello' do
    expect(subject).to match /Hello world/
    expect(subject).not_to match /Goodbye world/
    expect(subject).to eq 'Hello world!'
  end
end
```

In this example group we are using the `match` and `eq` matchers. Notice I didn't explicitly set a `subject` here, we're relying on the implicit subject set by the argument we passed to the `describe` method. The `match` matcher compares your subject with a regular expression, in this case `/Hello world/`. You can set a negative expectation by using the `not_to` method on our expectation. In this example, we want to make sure that we're saying "hello" so we want to make sure our subject does not match `/Goodbye world/`. Finally, we're using the `eq` matcher to compare our subject with the string `Hello world!`. Let's see what the output of this test would look like:

```
Hello world!
  should say hello

Finished in 0.00203 seconds (files took 0.07591 seconds to load)
1 examples, 0 failures
```

Notice we did not use the `be` matcher in our example. This is because the `be` matcher tests object identity while the `eq` matcher tests equality. Here's what the output would look like if we used the `be` matcher:

```
Hello world!
  should say hello (FAILED - 1)

Failures:

describe 'Hello world!' do
  1) Hello world! should say hello
     Failure/Error: expect(subject).to be 'Hello world!'

       expected #<String:70291920962560> => "Hello world!"
            got #<String:70291943850520> => "Hello world!"

       Compared using equal?, which compares object identity,
       but expected and actual are not the same object. Use
       `expect(actual).to eq(expected)` if you don't care about
       object identity in this example.
     # ./spec/blog_examples/hello_world_spec.rb:6:in `block (2 levels) in <top (required)>'

Finished in 0.01117 seconds (files took 0.07584 seconds to load)
1 examples, 1 failure

Failed examples:

rspec ./spec/blog_examples/hello_world_spec.rb:2 # Hello world! should say hello
```

As you can see, the `be` matcher actually compares the identity of each object instance, but this example uses two different instances of `String`. We're really only interested in comparing the values of the `String` instances so we use the `eq` matcher.

## Contexts

RSpec offers two primary methods of grouping tests together. We've already looked at one way, by creating **example groups** with the `describe` method. The other way is to use the `context` method. The `context` method is really just an alias for the `describe` method. The `describe` method is generally used to create example groups that describe an object, while the `context` method is used to create example groups that describe the state of an object that can vary. Here's an example describing an `Array` object with multiple contexts:

```ruby
describe Array do
  context 'with no elements' do
    let(:subject) { [] }
    it 'should be empty' do
      expect(subject.count).to eq 0
    end
  end

  context 'with elements' do
    let(:subject) { ['foo', 'bar', 'baz'] }
    it 'should not be empty' do
      expect(subject.count).not_to eq 0
    end
  end
end
```

Here we've grouped our examples by the class of object we're testing (`Array`), but we've segmented our examples based on the context we're looking to describe. Each `context` introduces a new scope. Examples grouped by `context` can have their own subjects, or they can inherit their subject from their parent. Here's an example where we inherit our subject from our parent and test for different contexts:

```ruby
describe Hash do
  let(:subject) {{ :foo => 'bar', :baz => baz_val }}
  let(:baz_val) { nil }

  it 'should have the foo key set to bar' do
    expect(subject[:foo]).to eq 'bar'
  end

  it 'should have the baz key set to nil' do
    expect(subject[:baz]).to be nil
  end

  context 'with baz_val set to qux' do
    let(:baz_val) { 'qux' }
    it 'should have the baz key set to qux' do
      expect(subject[:baz]).to eq 'qux'
    end
  end

  context 'with baz_val set to quux' do
    let(:baz_val) { 'quux' }
    it 'should have the baz key set to quux' do
      expect(subject[:baz]).to eq 'quux'
    end
  end
end
```

Here we're testing the same `Hash` object, but we're testing that object under different contexts. We're overriding the `baz_val` method inside our `context` blocks and ensuring that our object behaves the way we expect it to under different conditions. This allows us to test parts of our code that vary with context while avoiding duplicating our tests that focus on the parts of our code that remain stable. Segmenting our examples into `context` blocks makes our test code and output more readable. Here's the output of the above example:

```
Hash
  should have the foo key set to bar
  should have the baz key set to nil
  with baz_val set to qux
    should have the baz key set to qux
  with baz_val set to quux
    should have the baz key set to quux

Finished in 0.00189 seconds (files took 0.10379 seconds to load)
4 examples, 0 failures
```

You can declare a `context` outside of a `describe` block using the `shared_context` method. A **shared context** is a context that can be shared across many example groups. It's a way to describe common behavior without having to resort to duplicating code. We can break out the common behavior in our example above into a shared context:

```ruby
shared_context 'foo should always be set to bar' do
  it 'should have the foo key set to bar' do
    expect(subject[:foo]).to eq 'bar'
  end
end

describe Hash do
  let(:subject) {{ :foo => 'bar', :baz => baz_val }}
  let(:baz_val) { nil }

  include_context 'foo should always be set to bar'

  it 'should have the baz key set to nil' do
    expect(subject[:baz]).to be nil
  end

  context 'with baz_val set to qux' do
    let(:baz_val) { 'qux' }
    include_context 'foo should always be set to bar'
    it 'should have the baz key set to qux' do
      expect(subject[:baz]).to eq 'qux'
    end
  end

  context 'with baz_val set to quux' do
    let(:baz_val) { 'quux' }
    include_context 'foo should always be set to bar'
    it 'should have the baz key set to quux' do
      expect(subject[:baz]).to eq 'quux'
    end
  end
end
```

Since the `foo` key should always be set to `bar`, we can include this shared context to ensure that our `Hash` meets our expectations in all of our contexts.
Here's the output of that test:

```
Hash
  should have the foo key set to bar
  should have the baz key set to nil
  with baz_val set to qux
    should have the foo key set to bar
    should have the baz key set to qux
  with baz_val set to quux
    should have the foo key set to bar
    should have the baz key set to quux

Finished in 0.00221 seconds (files took 0.08721 seconds to load)
6 examples, 0 failures
```

Shared contexts are a powerful way to describe common behavior without resorting to boilerplate. Shared contexts can be stored in separate files and loaded using the `require` method at the beginning of your `spec` file.

# How does this relate to operations?

You're probably wondering when we're going to get to the "Ops" part of this blog post. Now that we've got a good understanding of some of the RSpec primitives, we'll dive into how these can be used when testing infrastructure. I'm going to use Serverspec and `rspec-puppet` for my examples. If you're not a Puppet user, don't worry, the two main Infracode testing tools (Beaker and Kitchen) both make heavy use of Serverspec.

There are two general types of tests that you'll write as an Infracoder: **acceptance** and **unit** tests. We'll use `rspec-puppet` for **unit** tests and `serverspec` for **acceptance** tests.

## Unit tests

Unit tests are written to test a specific bit of code. With `rspec-puppet` we test our Puppet *classes*, *defined types*, and *resources*. We'll use `rspec-puppet` to define a specification for a Puppet class. This class will install a package, place a configuration file, and start a service. Because we're using test **driven** development, we'll write our tests before we write the code. Let's get started:

```ruby
require 'spec_helper'

describe 'ntp' do
  it { is_expected.to compile }
end
```

Here we have written a test for the `ntp` class. We have a single example with the expectation that the catalog will compile with the `ntp` class included. There's some `rspec-puppet` magic going on here, so let's delve into this first. First, you'll notice that I'm not using an explicit subject here, we're relying on the implicit subject using the first argument passed to the `describe` method. The `rspec-puppet` helpers assume the first argument to the `describe` method is the name of a Puppet class. It will automatically generate a manifest with `include ntp` as the content and will use that manifest to compile a catalog. The resulting catalog object then gets stored as the `subject` for use in our examples. This test has a single expectation. Using the `compile` matcher, we set the expectation that the `catalog` will successfully compile. When we run this, we'll get an error:

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
  let(:params) {{
    :package_name => 'ntpd',
    :config_file  => '/etc/ntpd.conf',
    :service_name => 'ntpd'
  }}

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
  $package_name,
  $config_file,
  $service_name,
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

We've just refactored this code to be parameterized. Now we can talk about some testing best practices!
