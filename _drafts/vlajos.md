---
layout: single
modified: 2015-09-16
comments: true
tags: [apache, nginx, testing, http, wget, https, ssl]
title: "Verifying webserver configuration with some basic tests"
excerpt: "Behind firewalls, load balancers, cdns, reverse proxies and with a not totally simple webserver configuration, it can quite be hard to verify all the different locations, virtual hosts, aliases, rewrites."
categories: test
---

https://github.com/vlajos/vlajos.github.io/blob/master/_posts/2015-09-16-testing-webserver-http-headers.md

We have a project to restructure our webservers, consolidate their configuration, consider using new technologies and implement
best practices where it is possible.
Which means lots of configuration changes.
Our setup is not too complex. We have only a reverse proxy, 2 webservers (one of them is the reverse proxy), a firewall and we use sometimes
a CDN for different contents. Basically we serve 4 sites via https, we have a couple of aliases, redirects, rewrite rules and some
header manipulation for browser side caching, security and compression.

But if we count the different types of requests it is roughly

 * 4 (virtual hosts) *
 * 3 (caching logics) *
 * 2 (http redirects, https versions) *
 * 4-5 (aspects of the response we want to verify)

And a couple of security related rules.

So the number is higher than we would like to test it manually after each configuration change...

We googled but we could not find any tool which would offer any assertion mechanism for testing http headers.
To be honest I am a bit surprised, maybe we were not thorough enough.

Anyway we decided to implement something basic for ourselves which fulfils our needs and it is not more complex than the minimum.

Finally we picked `wget` and wrote a couple of trivial `bash` functions around it.

The main logic of the test "framework":

{% highlight bash %}
#!/bin/bash
wget(){
  export LASTRESULT=$(/usr/bin/wget -S --no-check-certificate --header="Accept-encoding: gzip" --max-redirect=0 --header="Host: $2" $1://$TargetIp:$3$4 -O /dev/null 2>&1)
  export LASTCALL="IP:$TargetIp $1://$2:$3$4"
}
testShouldContain(){
  echo "$LASTRESULT" |grep -q "$1" || echo "$LASTCALL should contain: '$1'"
}
testShouldNotContain(){
  echo "$LASTRESULT" |grep -q "$1" && echo "$LASTCALL should not contain: '$1'"
}
{% endhighlight %}

So it is around 10 lines and nothing super-fancy. (And uses global variables... Khm. It could be improved.)

But let's see it in action:

{% highlight bash %}
#!/bin/bash

export TargetIp=x.y.z.a
#export TargetIp=127.0.0.1

source test_framework.sh
wget_oursite(){
  wget https oursite 443 "$@"
}

# Host/port related redirects
wget http www.oursite.com 80 /
testShouldContain "HTTP/1.1 301 Moved Permanently"
testShouldContain 'Location: https://www.oursite.com/'
wget https oursite.com 443 /
testShouldContain "HTTP/1.1 301 Moved Permanently"
testShouldContain 'Location: https://www.oursite.com/'



{% endhighlight %}

So in around 2 hours we implemented 107 basic tests to verify our different assets, security, gzip, proxy and caching related rules.
Its runtime is around 2 secs.
So our main goal is achieved. If we change anything in the setup we can verify that with these rules roughly instantly.

Some more random comments:

 * The responses are cached. (At least until the following one.)
 * The asserts work with regular expressions. (see Expires: check)
 * The `wget` arguments are not 100% trivial so maybe they need some explanation:
   * `-S`: We started with `--debug` but it gave us too many unneeded lines. `-S` displays only the response headers.
   * `--no-check-certificate`: Unfortunately the server is not yet accessible under its final DNS name, so the url's host we are connecting to is differs from the name in the certificates.
   * `--header="Accept-encoding: gzip"`: We also want to test compression.
   * `--max-redirect=0`: By default wget would follow redirects, but in our case this is not required.
   * `--header="Host: $2"`: We want to test different virtual hosts and the current DNS settings pointing to the live system.
   * `$1://$TargetIp:$3$4`: This is only the construction of the URL.
   * `-O /dev/null`: Throw away the downloaded content.
   * `2>&1`: Merge wget's standard error into its output.
 * We could group the asserts with creating higher level functions. For example `testCacheHeaders`, `testGzipped` and so on.
   Maybe we will do this later.