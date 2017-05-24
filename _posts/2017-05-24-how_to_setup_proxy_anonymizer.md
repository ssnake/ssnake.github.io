---
layout: post
title: How to setup proxy anonymizer with SQUID
comments: true
tag: proxy, anonymizer, anonymous, squirt, nginx, linux, ubuntu
---

Recently Ukrainian President has imposed sanction on russian sites like vk.com, mail.ru and etc.
So, now all these web resources are unreachable from Ukraine. 

In general, for breaching the blocking of ip we can use VPN or Proxy. As for me proxy is more comfortable way to deal with the problem. You do not need put all your traffic via remote server. An browser extension could be installed to use proxy only for specific sites.

Below I'll describe a few methods how to setup proxy server on remote VPS server. For this we need a VPS server somewhere abroad, where web resources we instrested in are not blocked.
Suppose VPS has ubuntu installed.

Let's start!

## SQUID

First, we need to install it.

{% highlight bash %}
sudo apt-get install squid
{% endhighlight %}

Then make a copy of `squid.conf`

{% highlight bash %}
sudo mv /etc/squid3/squid.conf /etc/squid3/squid.conf.bak
{% endhighlight %}


Edit `squid.conf`

{% highlight bash %}
nano -w /etc/squid3/squid.conf
{% endhighlight %}

And copy paste the following text

{% highlight bash %}
http_port 0.0.0.0:3128                                                                                                                                                   

auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/passwd

acl foo proxy_auth REQUIRED                                                                                                                                              
http_access allow foo                                                                                                                                                    
http_access deny all 

forwarded_for off
request_header_access Allow allow all
request_header_access Authorization allow all
request_header_access WWW-Authenticate allow all
request_header_access Proxy-Authorization allow all
request_header_access Proxy-Authenticate allow all
request_header_access Cache-Control allow all
request_header_access Content-Encoding allow all
request_header_access Content-Length allow all
request_header_access Content-Type allow all
request_header_access Date allow all
request_header_access Expires allow all
request_header_access Host allow all
request_header_access If-Modified-Since allow all
request_header_access Last-Modified allow all
request_header_access Location allow all
request_header_access Pragma allow all
request_header_access Accept allow all
request_header_access Accept-Charset allow all
request_header_access Accept-Encoding allow all
request_header_access Accept-Language allow all
request_header_access Content-Language allow all
request_header_access Mime-Version allow all
request_header_access Retry-After allow all
request_header_access Title allow all
request_header_access Connection allow all
request_header_access Proxy-Connection allow all
request_header_access User-Agent allow all
request_header_access Cookie allow all
request_header_access All deny all
{% endhighlight %}

Let's take a look at this line

{% highlight bash %}
auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid3/passwd
{% endhighlight %}

It tells SQUID to use proxy authentication. Generally speaking we need to create ```passwd``` file with login/pass records from which SQUID will read and authenticate requests.

{% highlight bash %}
htpasswd -c passwd snake
sudo chown proxy:proxy passwd
sudo chmod u=r passwd
{% endhighlight %}

Lines below, tell SQUID to allow acl named ```foo``` to access proxy, because by default SQUID denies all requests

{% highlight bash %}
acl foo proxy_auth REQUIRED                                                                                                                                              
http_access allow foo                                                                                                                                                    
http_access deny all 
{% endhighlight %}

Now restart SQUID

{% highlight bash %}
sudo restart squid3
{% endhighlight %}


How to use proxy only for  blocked sites
========================

For example, I need to use direct connection for all internet except ```vk.com```.  Since I use ```Chrome``` I'll describe a method how to achive this using ```Chrome```. For other browsers you can use a same method


Install [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en-US)

Then go to to its options and then select ```auto switch``` profile. Add following lines
![pic]({{ "/img/switchy_omega.png" | prepend: site.baseurl }})

And now select `auto switch` profile
![pic]({{ "/img/switchy_omega2.png" | prepend: site.baseurl }})


### Enjoy!