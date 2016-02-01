---
layout: post
title: Running Ruby in production environments
date: 2014-03-23 21:02:00 BRT
tags: ruby, benchmark, passenger, puma, unicorn
categories: articles
redirect_from: /blog/2014/03/running-ruby-in-production-environments/
---
To run Ruby in production environments, I'll benchmark 3 of the widely used web-servers:

* Phusion Passenger (4.0.40)
* Unicorn (4.8.2)
* Puma (2.8.1)

These benchs are just for medium throughput applications, so I'm not comparing them with low load environments.

### Environment
Both of them using the same nginx (1.4.7) build as a front web-server to handle the slow clients and the same Ruby version: 2.1.1. Running on a Ubuntu 12.04, 3.4ghz quad-core processor and 4gb of RAM.

Also, I made sure all of them were using the same machine resources:

**Passenger:**
{% highlight nginx %}
passenger_max_pool_size 6;
passenger_min_instances 6;
passenger_pool_idle_time 0;
passenger_pre_start http://localhost:5001/;
passenger_max_request_queue_size 300;
{% endhighlight %}

**Unicorn:**
{% highlight ruby %}
worker_processes 6
preload_app true
{% endhighlight %}

**Puma:**
{% highlight ruby %}
workers 6
threads 1, 1
{% endhighlight %}

And to run the benchmarks, I used Apache Benchmark tool with 200 concurrent connections.

> **ab -n 10000 -c 200 http://localhost:5000/**

### Problems

The first one I tried was Puma. I must say this was the easiest one. I ran the tests and no errors troubled me.

However, when I set up Unicorn, I faced several problems. The same problems happened using Passenger. The benchs were running too fast and returned many "non 20x responses", so I did some research on what went wrong.

The first problem was that the nginx could't hit the Unicorn socket due to "_(11: Resource temporarily unavailable)_". I wasted a few hours and I finnaly get that the socket backlog was too low, therefore it was refusing new connections.

The solution is increasing the **_somaxconn_** directive in Kernel. We can do it changing the _**/etc/sysctl.conf**_ file, on the line **_net.core.somaxconn=65535_**, then you run _**sysctl -w**_ to update the setting without rebooting.

Also, be advised that in some cases you will have to increase your max open files as well.

But that's a tricky tuning. You should only increase this value if your hardware can handle it. Doesn't make sense increase this if your machine can't handle it, thus making your app slow. Maybe you should scale horizontally instead. As Unicorn documentation says, If you're doing extremely simple benchmarks and getting connection errors under high request rates, increasing your backlog above the already-generous default of 1024 can help avoid connection errors. Keep in mind this is not recommended for real traffic if you have another machine to failover to.

This is however, something I couldn't understand. Why did Puma ran just fine without any tuning? No errors were thrown, it's look like Puma manages the connection pool by itself, and keep incoming connections in the queue, I don't know!

Similar problem were faced with Passenger. The tests ran too fast and the errors were something like: The queue is full. I just increased the queue size to 300 (default is 100). Again, this is not something you should do in production unless you know what you're doing ;)

### Results

When I managed to resolve all these problems, I ran the benchmarks and... see by yourself.

**Passenger**

{% highlight rest %}
Server Software:        nginx/1.4.7
Server Hostname:        localhost
Server Port:            5001

Document Path:          /
Document Length:        3936 bytes

Concurrency Level:      200
Time taken for tests:   34.512 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      48150000 bytes
HTML transferred:       39360000 bytes
Requests per second:    289.75 [#/sec] (mean)
Time per request:       690.241 [ms] (mean)
Time per request:       3.451 [ms] (mean, across all concurrent requests)
Transfer rate:          1362.47 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.4      0       4
Processing:    44  684  37.3    683     915
Waiting:       43  683  37.3    682     909
Total:         47  684  37.1    683     918

Percentage of the requests served within a certain time (ms)
  50%    683
  66%    689
  75%    694
  80%    697
  90%    707
  95%    723
  98%    746
  99%    756
 100%    918 (longest request)
 {% endhighlight %}

**Unicorn**

{% highlight rest %}
Server Software:        nginx/1.4.7
Server Hostname:        localhost
Server Port:            5000

Document Path:          /
Document Length:        3936 bytes

Concurrency Level:      200
Time taken for tests:   33.291 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      47620000 bytes
HTML transferred:       39360000 bytes
Requests per second:    300.38 [#/sec] (mean)
Time per request:       665.815 [ms] (mean)
Time per request:       3.329 [ms] (mean, across all concurrent requests)
Transfer rate:          1396.90 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.5      0       7
Processing:    21  659  52.6    664     722
Waiting:       16  659  52.6    664     722
Total:         25  659  52.2    664     722

Percentage of the requests served within a certain time (ms)
  50%    664
  66%    670
  75%    673
  80%    675
  90%    681
  95%    686
  98%    693
  99%    698
 100%    722 (longest request)
 {% endhighlight %}

**Puma**

{% highlight rest %}
Server Software:        nginx/1.4.7
Server Hostname:        localhost
Server Port:            5000

Document Path:          /
Document Length:        3936 bytes

Concurrency Level:      200
Time taken for tests:   36.170 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      47460000 bytes
HTML transferred:       39360000 bytes
Requests per second:    276.47 [#/sec] (mean)
Time per request:       723.396 [ms] (mean)
Time per request:       3.617 [ms] (mean, across all concurrent requests)
Transfer rate:          1281.39 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   1.7      0      16
Processing:    12  701 767.4    437    3433
Waiting:       11  700 767.4    436    3432
Total:         12  701 767.7    437    3433

Percentage of the requests served within a certain time (ms)
  50%    437
  66%    670
  75%    947
  80%   1224
  90%   1823
  95%   2419
  98%   3081
  99%   3194
 100%   3433 (longest request)
 {% endhighlight %}

### Conclusion
I don't think that performance is a reason to change your webserver. Consider other options like community, features and ease to use.

Today I run a large application with Passenger and there's some reasons I want to change, and I'm happy to see that performance isn't a concern anymore.

Also, I have to figure out why Puma works so seamlessly, while Passenger and Unicorn complained about the maximum socket connections. Is this a good thing? I'll certainly share when I find out ;)

### Resources

* <http://whowish-programming.blogspot.com.br/2011/10/nginx-502-bad-gateway.html>{:target="blank"}
* <http://askubuntu.com/questions/162229/how-do-i-increase-the-open-files-limit-for-a-non-root-user>
* <http://forum.nginx.org/read.php?11,215606,235395>{:target="blank"}
* <http://www.modrails.com/documentation/Users%20guide%20Nginx.html>{:target="blank"}
* <http://unicorn.bogomips.org/TUNING.html>{:target="blank"}
* <http://ubuntuforums.org/showthread.php?t=1198281>{:target="blank"}
