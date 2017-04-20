---
layout: post
title:  "Benchmarking in Ruby"
date:   2017-04-19 12:57:00 +1000
categories: ruby benchmarking docker
---
We had an interesting problem. Reading and detecting files that matched a certain pattern seemed slow. We thought it might be fun to benchmark some solutions and look at the results.

First we created 100 folders each with 100 files.
{% highlight ruby %}
# Make 100 files in 100 folders
(0..100).to_a.each do |number|
  `mkdir #{number}`
  `touch #{number}/file{1..100}`
end
{% endhighlight %}

Then we wrote a script to test how fast different solutions were:
{% highlight ruby %}
def current_directory_search
  Dir["*0/"]
end

def current
  files = []
  list = current_directory_search
  list.each do |dir|
    Dir.foreach(dir) do |file|
      files << "#{dir}#{file}" if /5$/.match(file)
    end
  end
  files
end

def new_directory_search
  Dir.entries(".").select {|x| x =~ /0$/ }.map {|x| "#{x}/"}
end

def new
  files = []
  list = new_directory_search
  list.each do |dir|
    files << Dir.entries(dir).select {|x| x =~ /5$/}.map {|x| "#{dir}#{x}"}
  end
  files.flatten
end

def mixed
  files = []
  list = new_directory_search
  list.each do |dir|
    Dir.foreach(dir) do |file|
      files << "#{dir}#{file}" if /5$/.match(file)
    end
  end
  files
end

TIMES = 500
require 'benchmark'
Benchmark.bm(43) do |x|
  x.report("      current directory search:") { TIMES.times { current_directory_search } }
  x.report("          new directory search:") { TIMES.times { new_directory_search } }
  x.report("   current dir and file search:") { TIMES.times { current } }
  x.report("new dir search new file search:") { TIMES.times { new } }
  x.report("new dir search old file search:") { TIMES.times { mixed } }
end
{% endhighlight %}

In particular, we were interested in seeing if using Dir globbing was faster than doing a Regex match across all the files.
{% highlight shell %}
      current directory search:  0.110000   0.370000   0.480000 (  4.731889)
          new directory search:  0.090000   0.110000   0.200000 (  1.933240)
   current dir and file search:  1.310000   1.230000   2.540000 ( 35.785127)
new dir search new file search:  1.110000   1.040000   2.150000 ( 31.571014)
new dir search old file search:  1.280000   1.290000   2.570000 ( 33.142984)
{% endhighlight %}

We were a bit surprised that using the Regex was faster, than Dir globbing, but even more surprised that we
didn't get much of a saving when we applied to locating files.

The biggest thing to come out of this for myself though, was how easy it was to benchmark this.
Ruby's standard library makes it really easy!
