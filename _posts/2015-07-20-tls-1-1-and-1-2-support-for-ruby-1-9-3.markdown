---
layout: post
title: "TLS 1.1 and 1.2 support for Ruby 1.9.3"
date: 2015-07-20 10:36:00 BRT
tags: ruby, tls
categories: articles
redirect_from: /blog/2015/07/tls-1-1-and-1-2-support-for-ruby-1-9-3/
---
As of the begining of this year, precisely at February 23th, Ruby 1.9.3 is no longer supported: Not even for security or bug fixes. The recommendation is to upgrade to Ruby 2.0.0 or above - even though 2.0.0 has a limited support until Feb 24th 2016.

However, the migration is not as seamlessly as it looks, and some applications just can't be upgraded overnight. And due to some recent SSL security bugs, some websites stopped providing support to SSLv2 and SSLv3, or even TLSv1.

Long story short, I created this patch to add support to TLS 1.1 and TLS 1.2 to Ruby 1.9.3 to those who just can't upgrade right away.

The patch is available at: <https://gist.github.com/kriansa/dd1b9a0d8dfec776fc91>{:target="blank"}

To install it you'll have to recompile Ruby, but using [rvm](http://rvm.io){:target="blank"} is not that hard at all:

{% highlight bash %}
$ rvm install 1.9 --patch https://gist.githubusercontent.com/kriansa/dd1b9a0d8dfec776fc91/raw/4065d5a08e75e1f071dadeabf5a85b43de6bfad4/openssl_tls_1.2.patch
{% endhighlight %}

Have fun with you outdated Ruby :)
