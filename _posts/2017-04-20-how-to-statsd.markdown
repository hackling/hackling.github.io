---
layout: post
title:  "How to test StatsD"
date:   2017-04-20 12:57:00 +1000
categories: statsd docker
---
{% highlight shell %}
  while :; do nc -l -u 8125; done
{% endhighlight %}

This basically listens on port 8125, the default StatsD port, and outputs the content.
This is a handy way if you want to test StatsD locally. Just set the statds reporting URL to localhost:8125
