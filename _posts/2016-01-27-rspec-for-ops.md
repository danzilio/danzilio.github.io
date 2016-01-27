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

I've noticed a lot of interest in RSpec from ops folks lately, likely driven by infracode testing tools like [Kitchen](http://kitchen-ci.org), [Beaker](https://github.com/puppetlabs/beaker), and [Serverspec](http://serverspec.org). I've also noticed a lot of ops folks using `rspec` without really understanding some of the fundamentals. This results in some suboptimal `rspec` code. This blog post is an attempt to outline some of the basics of test driven development with `rspec` from the perspective of an Op :) I'll start by outlining `rspec` basics followed by some (hopefully) relatable examples.

# RSpec Basics

RSpec is a behavior driven development (BDD) framework for Ruby. BDD focuses on the behavior and expected outcomes of your code. Popular BDD frameworks use simple domain-specific languages that employ natural-lanaguage constructs. As a result, BDD code is often very easy to read with inputs and expectations being very easy to identify. RSpec offers several primitives to model the beavior of your application including **groups**, **examples**, **contexts**, and **matchers**.

## Examples and Groups

The most fundamental primitive in `rspec` is the **example**. An example is the description of a behavior you expect your code to exhibit. Examples are declared using the `it` method and are often grouped together logically inside a (very creatively named) **example group**. Example groups are declared with the `describe` method. Example groups make up your application's specification; they often end up looking a lot like lists of acceptance criteria. Here's a simple example:

```ruby
it { is_expected.to be true }
```

This code can be read as "it is expected to be true" and literally means that the subject of the test (we'll go into more detail in a minute) should return `true`. I didn't set a `subject` in the example above, but you can do that with the `let` method:

```ruby
let(:subject) { true }

it { is_expected.to be true }
```

The `let` method creates a [memoized](https://en.wikipedia.org/wiki/Memoization) helper method set to the value passed in the block. A memoized helper method is basically an `attr_accessor` on an instance variable. Here's what the `let` method does under the hood (basically):

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

Here we have an example group named `Truthiness`, we've set the `subject` to `true`, and we've created an example that expects our subject to be `true`. Let's run this code and see what the output is:

```
Truthiness
  should equal true

Finished in 0.00107 seconds (files took 0.08412 seconds to load)
1 example, 0 failures
```

## Expectations

An example is just a list of expected behaviors. Our example has one **expectation**, that the subject should equal `true`. Our example above could also be expressed using the `expect` method:

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

As you can see, examples can have any number of expectations. You should use as many expectations as necessary to validate the behavior you're testing in an example.
