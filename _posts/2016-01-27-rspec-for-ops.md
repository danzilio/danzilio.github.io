---
layout: post
title: "RSpec For Ops: Essentials"
date: 2016-01-27
tags:
  - RSpec
  - TDD
  - Testing
comments: true
published: false
---

# RSpec for Ops: Essentials

I've noticed a lot of interest in RSpec from ops folks lately, likely driven by infracode testing tools like [Kitchen](http://kitchen-ci.org), [Beaker](https://github.com/puppetlabs/beaker), and [Serverspec](http://serverspec.org). I've also noticed a lot of ops folks using `rspec` without really understanding some of the fundamentals. This results in some suboptimal `rspec` code, and generally lots of headaches for newcomers to `rspec`. This blog post is an attempt to outline some of the basics of test driven development with `rspec` from the perspective of an Op :) I'll start by outlining `rspec` basics, another post will follow with some (hopefully) relatable examples.

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

# Making the connection

This post focused mostly on RSpec itself. In a future post I'll bring these elements together with some examples using `rspec-puppet` and `serverspec`.
