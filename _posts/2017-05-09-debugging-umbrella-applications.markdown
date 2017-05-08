---
layout: post
title:  "How to debug an umbrella application"
date:   2017-05-09 12:57:00 +1000
categories: elixir umbrella debugging
---
We run our applications in Docker, and sometimes, you've just got to go on Staging or Production to debug and issue.
Containerization helps us abstract a bunch of cool things away, but there's a couple problems with running `docker run application` when it has all the configuration that your running applications have.

In particular, the big one, is background tasks. If run `iex -S mix` and it boots your background worker jobs up, you might get some unexpected behaviour, especially if your code is written to only expect one worker at a time.

In order to start an `iex` session, with your mix application, but not starting any of the children, you want to run:

{% highlight shell %}
iex -S mix run --no-start
{% endhighlight %}


This gives you access to all the code, but doesn't run anything.
In my case, I wanted access to the child application that handles the database sides of things, so once in `iex` I ran the following:

{% highlight shell %}
iex(1)> Application.ensure_all_started(:panoramix_ecto)
{% endhighlight %}

where `panoramix_ecto` is the name of the application.


With all this, I could safely poke around the database in staging, verify the schemas were casting as expected, all without running any of the other processes of the umbrella application.
