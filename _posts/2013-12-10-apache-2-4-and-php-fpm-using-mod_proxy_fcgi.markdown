---
layout: post
title: Apache 2.4 and PHP-FPM using mod_proxy_fcgi
date: 2013-12-10 17:01:00 BRT
tags: apache, php
categories: articles
---
Recently I needed to install the PHP environment on my production server. I wanted to try the new Apache 2.4 features, so I'm gonna bring you my experience deploying this server.

![apache](http://lucene.apache.org/images/mantle-asf.png)

There's many differences between the 2.2 and the 2.4, so if you want to dive into it, you should check it at <a href="http://httpd.apache.org/docs/current/upgrading.html">http://httpd.apache.org/docs/current/upgrading.html</a>.

For performance reasons, and I won't explain it here, we should use PHP-FPM, which is the <em><strong>PHP FastCGI Process Manager</strong></em> (it's important to know why it's called so). In other words, is a PHP background service that runs script in demand (more info: <a href="http://php-fpm.org/about/">http://php-fpm.org/about/</a>).

There's two ways to communicate with Apache: using either <em>mod_fastcgi</em> or <em>mod_proxy_fcgi</em>. You may ask why I'm not mentioning mod_fcgi. That's because mod_fcgi doesn't support spawned CGI servers, which is bad because PHP-FPM is managed by itself, not by Apache, so mod_fcgi isn't compatible.

The problem with <em>mod_fastcgi</em> is that it's pretty old (its last update was in 2007) and compiling it for is kinda difficult. I had to change its source code to compile it.

And finally, <em>mod_proxy_fcgi</em> remains, which is actively developed and bundled with Apache, you just have to enable it.

## Configure _mod_proxy_fcgi_

Disclaimer: At this point, I'm assuming you already installed Apache 2.4 and the PHP-FPM, and also have some experience with Apache, VirtualHosts, mod_rewrite and such.

There's not much to configure in mod_proxy_fcgi. Once you enable it, a new <em>wrapper</em> is enabled, "<em>fcgi</em>" (just like "<em>http</em>"), and you can use it in any directive, such as <a title="mod_proxy" href="http://httpd.apache.org/docs/2.4/mod/mod_proxy.html" target="_blank">ProxyPass</a>, <a title="mod_proxy" href="http://httpd.apache.org/docs/2.4/mod/mod_proxy.html" target="_blank">ProxyPassMatch</a>, <a title="mod_rewrite" href="http://httpd.apache.org/docs/2.4/mod/mod_rewrite.html" target="_blank">RewriteRule</a> and so on.

With this wrapper, you can use any running CGI server easily. Lets try it with a simple redirect any request to a front file, which handles it and displays the correct page (most MVC framework do it).

Something you should know about <em>mod_proxy_fcgi</em> is that currently it doesn't support UNIX sockets, so you must start your PHP-FPM process using a TCP port, which is default when you install it.

> RewriteEngine On<br />
> RewriteCond /home/user/site/public%{REQUEST_URI} !-s<br />
> RewriteCond /home/user/site/public%{REQUEST_URI} !-l<br />
> RewriteRule ^.*$ fcgi://localhost:4000 /home/user/site/public/index.php [P,NC,L]

Notice that I used the <strong>P</strong> flag, which means "use proxy redirection" instead common URL. These rules will check if the REQUEST_URI string is not a file or a symlink in that folder (/home/user/site/public), and if it's not, it will be handled by the index.php file.

Yes, this simple. With this example you can solve almost any problem with MVC frameworks.

Well, it took me all afternoon to learn all these little details, so I hope you enjoy and get it faster than I did :P

## References

* <http://wiki.apache.org/httpd/PHP-FPM>
* <http://www.spinics.net/lists/apache-users/msg102465.html>
* <http://httpd.apache.org/docs/2.4/mod/mod_rewrite.html>
* <http://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html>
* <http://httpd.apache.org/docs/2.4/mod/mod_proxy.html>
