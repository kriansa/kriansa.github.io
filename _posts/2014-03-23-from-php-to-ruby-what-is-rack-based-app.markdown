---
layout: post
title: "From PHP to Ruby: What is a Rack-based app?"
date: 2014-03-12 19:42:00 BRT
tags: php, ruby, rack, web, from-php-to-ruby
categories: articles
---
I want to start this blog series called "_From PHP to Ruby_", which are intended to be small articles on how different is one from another and such. I hope somebody can learn something ;)

If you're new in Ruby world and came from, not just PHP, but every language, you may notice some technical terms around it. In the beginning, I was pretty confused about "_Rack-based_" stuff. So I'm gonna try to explain what I've learned from it.

In the beginning, the way every application was exposed to the web was through <a href="http://en.wikipedia.org/wiki/Common_Gateway_Interface">CGI</a>. So I could write a C application and using a webserver with CGI support, it could communicate with web.

However, this method has many cons: each framework or *webapp* has to write the basic web flow structure: **Request**, **Response**, **Params**, **Cookies**, **Environment** and so on. And therefore, many projects often wrote the code that solved the same problem: parsing the Http request. Pretty much what PHP's Symfony calls <a href="http://symfony.com/doc/current/components/http_foundation/introduction.html">HttpFoundation</a>. Today PHP-FIG has a standard for this so called Message Interface, [PSR7](http://www.php-fig.org/psr/psr-7/).

So, someday Ryan Allen <a href="http://yeahnah.org/files/rack-presentation-oct-07.pdf">came up with the idea</a> of creating a simple API interface that sits between the **webserver** and the **webapplication** (thus it's a 'middleware') and called it <a href="http://rack.github.io/">Rack</a>.

Differently than PHP's similar projects that are different for almost every framework, this middleware became very popular and widely supported, so today almost every Ruby webserver and pretty every Ruby web framework also supports it.

Writing a simple Rack application it's really easy, all you have to write is a a lambda like so:

{% highlight ruby %}
run lambda { [200, {"Content-Type" => "text/html"}, "Hello Rack!"] }
{% endhighlight %}

That's pretty much it! :)
