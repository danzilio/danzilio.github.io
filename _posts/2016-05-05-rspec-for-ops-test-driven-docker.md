---
layout: post
title: "RSpec For Ops Part 4: Test driven Docker"
date: 2016-05-05
tags:
  - RSpec
  - RSpec for Ops
  - TDD
  - Testing
  - blogmonth
  - 30in30
  - blog month 2016
  - Docker
comments: true
---

This is the fourth part in a series of blog posts on RSpec for Ops. See the
first post
[here](http://blog.danzil.io/2016/02/05/rspec-for-ops-part-1-essentials.html),
second post
[here](http://blog.danzil.io/2016/05/03/rspec-for-ops-rspec-puppet.html), and
third post
[here](http://blog.danzil.io/2016/05/04/rspec-for-ops-test-driven-design.html).

Over the course of this blog series I've talked about RSpec in general and TDD
with `rspec-puppet`. Now I'd like to take a more platform-agnostic approach by
exploring TDD with [Serverspec](http://serverspec.org). For this particular
example look at how to build a test driven Docker image using Serverspec.

## Serverspec
Serverspec is a framework built on top of RSpec that allows you to write tests
that examine the state of a running system. Serverspec provides a number of
cross-platform matchers and helpers that build on the RSpec DSL. This enables us
to examine a system's resources and ensure that the system's state matches our
expectations. Let's take a look at a simple Serverspec test.

```ruby
describe 'a web server' do
  it 'should be installed and running' do
    expect(package('apache2')).to be_installed
    expect(process('apache2')).to be_running
    expect(port('80')).to be_listening
  end

  describe file('/etc/apache2/sites-enabled/000-default.conf') do
    it { is_expected.to be_symlink }
    its(:content) { is_expected.to match /DocumentRoot \/var\/www\/html/ }
  end
end
```

In this snippet we have an example group describing a web server. We've written
examples that test to make sure the `apache2` package is installed, that the
`apache2` process is running and listening on port 80, that the default virtual
host is enabled, and the document root is configured correctly.

Serverspec gives us several options for executing these tests. We can run these
tests on the local system, we can use `ssh` to connect to a remote system, or we
can run these commands inside a Docker container. There are many other execution
backends, but these are the most common. Let's take a look at what our
`spec_helper.rb` would look like (we're using the `docker-api` gem).

```ruby
require 'serverspec'
require 'docker'

set :backend, :docker

project_root = File.expand_path(File.join(__FILE__, '..', '..', 'docker'))

RSpec.configure do |c|
  c.before(:suite) do
    set :docker_image, Docker::Image.build_from_dir(project_root).id
  end
end
```

This `spec_helper.rb` file sets up all of our testing dependencies, including
loading the `serverspec` and `docker` libraries we'll be using in all of our
tests. This file also tells Serverspec that we're using the `docker` backend,
sets the `project_root` variable, and configures a `before` hook for our RSpec
tests.

A note on `Dockerfile` builds. Builds with a `Dockerfile` are **idempotent**,
meaning that the Docker container will only be rebuilt when a change is made to
the `Dockerfile`. This allows us to run these tests frequently and with
relatively low overhead. Here, we've configured the `before` hook to build our
Docker container before running our tests. You'll notice that we're using the
`suite` before hook, because we only want to build the Docker container once
during each run of Serverspec.

In order to run these tests, we need a minimal Dockerfile. Remember, we wrote
the tests first, so we want to run them and make sure they fail. Let's create a
minimal Dockerfile:

```dockerfile
FROM ubuntu:14.04
```

This is the absolute bare minimum necessary to run these tests. This
`Dockerfile` only sets the parent image to `ubuntu:14.04`. Let's run our tests
and see what we get:

```
a web server
  should be installed and running (FAILED - 1)
  File "/etc/apache2/sites-enabled/000-default.conf"
    should be symlink (FAILED - 2)
    content
      should match /DocumentRoot \/var\/www\/html/ (FAILED - 3)

Failures:

  1) a web server should be installed and running
     Failure/Error: expect(package('apache2')).to be_installed
       expected Package "apache2" to be installed

     # ./spec/acceptance/apache_spec.rb:5:in `block (2 levels) in <top (required)>'

  2) a web server File "/etc/apache2/sites-enabled/000-default.conf" should be symlink
     Failure/Error: it { is_expected.to be_symlink }
       expected `File "/etc/apache2/sites-enabled/000-default.conf".symlink?` to return true, got false

     # ./spec/acceptance/apache_spec.rb:11:in `block (3 levels) in <top (required)>'

  3) a web server File "/etc/apache2/sites-enabled/000-default.conf" content should match /DocumentRoot \/var\/www\/html/
     Failure/Error: its(:content) { is_expected.to match /DocumentRoot \/var\/www\/html/ }
       expected "" to match /DocumentRoot \/var\/www\/html/
       Diff:
       @@ -1,2 +1,2 @@
       -/DocumentRoot \/var\/www\/html/
       +""


     # ./spec/acceptance/apache_spec.rb:12:in `block (3 levels) in <top (required)>'

Finished in 2.08 seconds (files took 0.34553 seconds to load)
3 examples, 3 failures

Failed examples:

rspec ./spec/acceptance/apache_spec.rb:4 # a web server should be installed and running
rspec ./spec/acceptance/apache_spec.rb:11 # a web server File "/etc/apache2/sites-enabled/000-default.conf" should be symlink
rspec ./spec/acceptance/apache_spec.rb:12 # a web server File "/etc/apache2/sites-enabled/000-default.conf" content should match /DocumentRoot \/var\/www\/html/
```

Great! We have our failing tests. Now, let's implement those features in our
`Dockerfile`. Remember, we want to write the _minimum amount of code_ necessary
to get our tests to pass.

```
FROM ubuntu:14.04
RUN apt-get update && apt-get install -y apache2
CMD apache2ctl -D FOREGROUND
```

As you can see, we've edited our `Dockerfile` to install the `apache2` package, and configured the web server to start with the container. Now let's run our tests again and see how things look.

```
a web server
  should be installed and running
  File "/etc/apache2/sites-enabled/000-default.conf"
    should be symlink
    content
      should match /DocumentRoot \/var\/www\/html/

Finished in 13.85 seconds (files took 0.4131 seconds to load)
3 examples, 0 failures
```

Awesome! We have a working Dockerized web server, and it's WEBSCALE! Now you're
ready to conquer the internet. If you're interested in using this example for
your own Docker development purposes, you can find the code on my GitHub
[here](https://github.com/danzilio/TDDocker). Happy containerizing!
