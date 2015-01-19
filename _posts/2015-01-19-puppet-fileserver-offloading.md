---
layout: post
title: "Puppet Fileserver Offloading"
date: 2015-01-19
tags:
  - Puppet Scalability
  - Constant Contact
  - Puppet 2.7
  - Upgrade
  - Puppet
comments: true
published: true
---

I haven't written much about our Puppet environment at Constant Contact since I started there just about a year ago; I was brought in to help with the transition to Puppet 3. A bit of information about our environment: we have a couple hundred developers committing to our Puppet code, with about 500 commits per week to our main Puppet repository which contains over 100,000 lines of (mostly custom) code. As for scale, we have five distinct deployment environments across three datacenters; in total we have about 6,000 servers. Each environment hosts a pair of Puppet masters in an active-active HA configuration behind a load balancer.

Constant Contact was a very early adoper of Puppet, and it really shows in the code. We began using Puppet around version 0.24.8; well before paramterized classes would be introduced in version 2.6. As a result, our legacy code makes copious use of dynamic scope. Mostly due to inertia and resource constraints, we've been stuck on Puppet 2.7 (we should be on 3.7 by April, posts forthcoming). Puppet 2.7 has a lot of disadvantages for us especially given how fast our environment and the community have moved around it. The version of Puppet we're stuck on has a lot of bugs and is not particularly performant. To move to Puppet 3 we knew we needed to train hundreds of developers and refactor a ton of code, which meant that the transition away from Puppet 2.7 would take a long time. At the same time, lengthy Puppet run times became intolerable to maintain velocity with our Continuous Deployment efforts.

We needed a way to reduce Puppet run times with Puppet 2.7. Since we're a fairly data-driven bunch, our first step was to gather some performance data. A quick look at our Puppet report data told us that the File resource was responsible for over 30% of our Puppet run time. Looking for more granular information, we configured Logstash to watch our Apache logs and send the API endpoint response times to Graphite. A trend quickly emerged; the median response time for the `file_metadata` endpoint was over 1 second (!!). In our environment, the `file_metadata` endpoint is the most frequently hit endpoint. An average Puppet run hits this endpoint well over a hundred times. Enter Puppet 3.

We've been preparing for Puppet 3 for the past year or so. Around this time we began standing up pairs of Puppet 3 masters in each environment, similar to our Puppet 2.7 setup. Initial tests confirmed the fact that Puppet 3 is indeed **much** faster than Puppet 2.7 (this shouldn't be a surprise to anybody). Mostly as a thought experiment, I decided to experiment with proxying the `file_metadata` and `file_content` endpoints to our Puppet 3 masters in our development environment. The results were pretty interesting. Here's a snapshot from our Grafana dashboard showing the median response times for the `file_metadata` API endpoints.

![](/assets/offloading_results.png)

As you can see, our API response times dropped from upwards of 1 second to under 200ms for the `file_metadata` endpoints. After exhaustive testing, we found that this reduced our Puppet run times by about a third (!!). Our CD team was very excited to see their Jenkins jobs speeding up. We've implemented this offloading configuration across all environments.

Here's what the architecture looks like:

<div align='center'>
{% mermaid %}
graph TB;
	cl{Clients}-- https -->lb1(Front End Balancer);
	lb1-- https -->m1[Puppet 2.7 Master<br /><code>master1.example.com</code>];
	lb1-- https -->m2[Puppet 2.7 Master<br /><code>master2.example.com</code>];
	m1-- http -->lb2(Back End Balancer);
	m2-- http -->lb2;
	lb2-- http -->fs1[Puppet 3.7 Master<br /><code>master3.example.com</code>];
	lb2-- http -->fs2[Puppet 3.7 Master<br /><code>master4.example.com</code>];
{% endmermaid %}
</div>

The Puppet 2.7 clients connect to the Puppet masters via the `Front End Balancer` on the standard port. The load balancer sends the connection to one of the two Puppet 2.7 masters. From there, we use Apache to proxy all traffic headed for the `file_content`, `file_metadata`, or `file_metadatas` endpoints to the `Back End Balancer` on port 18141 which balances this traffic among the back end file servers.

Let's take a look at some of the configurations we used to make this happen:

## Front End Configuration

Here's the Apache configuration file for the Puppet 2.7 Masters (of particular note are lines 35-42):

{% highlight apache linenos %}
Listen 8140
<VirtualHost *:8140>
  SSLEngine on
  SSLProtocol all -SSLv2 -SSLv3
  SSLCipherSuite ALL:!aNULL:!eNULL:!DES:!3DES:!IDEA:!SEED:!DSS:!PSK:!RC4:!MD5:+HIGH:+MEDIUM:!LOW:!SSLv2:!EXP

  SSLCertificateFile      /var/lib/puppet/ssl/certs/master1.example.com.pem
  SSLCertificateKeyFile   /var/lib/puppet/ssl/private_keys/master1.example.com.pem
  SSLCertificateChainFile /var/lib/puppet/ssl/ca/ca_crt.pem
  SSLCACertificateFile    /var/lib/puppet/ssl/ca/ca_crt.pem
  SSLCARevocationFile     /var/lib/puppet/ssl/ca/ca_crl.pem

  SSLVerifyClient optional
  SSLVerifyDepth  1
  SSLOptions +StdEnvVars +ExportCertData

  RequestHeader set X-SSL-Subject %{SSL_CLIENT_S_DN}e
  RequestHeader set X-SSL-Client-DN %{SSL_CLIENT_S_DN}e
  RequestHeader set X-Client-Verify %{SSL_CLIENT_VERIFY}e

  SetEnvIf X-SSL-Subject "(.*)" SSL_CLIENT_S_DN=$1
  SetEnvIf X-Client-Verify "(.*)" SSL_CLIENT_VERIFY=$1
  SetEnvIf X-Forwarded-For "(.*)" REMOTE_ADDR=$1

  RackAutoDetect On
  DocumentRoot /etc/puppet/rack/public/
	
  <Directory /etc/puppet/rack/>
    Options None
    AllowOverride None
    Order allow,deny
    allow from all
  </Directory>

  ProxyPreserveHost On
  ProxyPassMatch ^(/.*?)/file_(.*)/(.*)$ balancer://puppetfileserver
  ProxyPassReverse ^(/.*?)/file_(.*)/(.*)$ balancer://puppetfileserver

  <Proxy balancer://puppetfileserver>
    BalancerMember http://master3.example.com:18141 ping=5 disablereuse=on retry=5 ttl=120
    BalancerMember http://master4.example.com:18141 ping=5 disablereuse=on retry=5 ttl=120
  </Proxy>

  ErrorLog  /var/log/httpd/puppet_error.log
  LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" api
  CustomLog /var/log/httpd/puppet_access.log api
</VirtualHost>
{% endhighlight %}

We're unable to manage the Apache configuration on our 2.7 masters with the `puppetlabs/apache` module (it doesn't work with the version of 2.7 we're stuck on), otherwise I would've provided a code snippet (see below for the Puppet 3 masters). It should be fairly easy to construct an `apache::vhost` definition from the configuration above.

## Back End Configuration

Here's the `apache::vhost` definition for the backend file servers.

```ruby
apache::vhost { "${::fqdn}-fileserver":
  port              => 18141,
  serveraliases     => $::fqdn,
  docroot           => '/etc/puppet/rack/public/',
  directories       => [{
    path            => '/etc/puppet/rack/',
    options         => 'None',
    allow_override  => 'None',
    order           => 'Allow,Deny',
    allow           => ['from master1.example.com', 'from master2.example.com'],
  }],
  setenvif          => [
    'X-Client-Verify "(.*)" SSL_CLIENT_VERIFY = $1',
    'X-SSL-Client-DN "(.*)" SSL_CLIENT_S_DN   = $1',
  ],
  custom_fragment   => '  PassengerEnabled on',
}
```

Which results in this Apache configuration file:

{% highlight apache %}
# ************************************
# Vhost template in module puppetlabs-apache
# Managed by Puppet
# ************************************

<VirtualHost *:18141>
  ServerName master4.example.com-fileserver

  ## Vhost docroot
  DocumentRoot "/etc/puppet/rack/pubilc/"

  ## Directories, there should at least be a declaration for /etc/puppet/rack/pubilc/

  <Directory "/etc/puppet/rack/">
    Options None
    AllowOverride None
    Order Allow,Deny
    Allow from master1.example.com
    Allow from master2.example.com
  </Directory>

  ## Logging
  ErrorLog "/var/log/httpd/master4.example.com-fileserver_error.log"
  ServerSignature Off

  ## Server aliases
  ServerAlias master4.example.com
  SetEnvIf X-Client-Verify "(.*)" SSL_CLIENT_VERIFY=$1
  SetEnvIf X-SSL-Client-DN "(.*)" SSL_CLIENT_S_DN=$1

  ## Custom fragment
  PassengerEnabled On
</VirtualHost>
{% endhighlight %}

This was a relatively small amount of work for a huge payoff. We've been able to meet the demand for quicker Puppet runs, while allowing us to take the time we need to have a measured transition to Puppet 3. The plan is to simply point our clients at the new master cluster once the codebase is updated. I'll write a lot more on how we managed to transition. Hoepfully the upgrade to Puppet 4 will happen much more quickly (I'm targeting the end of CY15 for that).

Is this interesting/helpful? Can you think of a better way to do this? Am I the only one doing something like this?! Let me know!
