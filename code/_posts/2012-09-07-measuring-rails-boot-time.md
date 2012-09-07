---
layout: post
title: Measuring Rails Boot Time
---

Our main rails app is pretty slow to load, so I hacked together a really
simple report to show what's going on during boot. This system could use
a lot of improvement to aggregate together requires, but it's enough to
draw some immediate conclusions.

First, I created this script that overrides `Kernel.require`:

{% highlight ruby %}
Kernel.require 'singleton'
class BootMeasurement
  include Singleton
  def initialize
    @req_starts = {}
    @req_times = {}
  end

  def self.start(requirement)
    instance.start(requirement.to_s)
  end
  def self.finish(requirement)
    instance.finish(requirement.to_s)
  end

  def start(req)
    @first ||= Time.now
    @req_starts[req] ||= []
    @req_starts[req] << Time.now
  end
  def finish(req)
    raise "Requirement #{req} unstarted!" unless @req_starts.include? req
    @req_times[req] ||= []
    @req_times[req] << Time.now - @req_starts[req].last
    @last = Time.now
  end

  def self.report
    instance.report
  end

  def self.fmt(time)
    "#{time.round(2)} sec"
  end

  def report
    return unless @req_times.any?
    lng = @req_times.keys.max{|a,b| a.length <=> b.length }.length
    @req_times.map do |req, times|
      ttime = times.inject(0){|s, e| s + e }
      [req, ttime]
    end.sort{|a,b| a[1] <=> b[1] }.each do |req, ttime|
      puts "#{req.ljust(lng, ' ')}: #{BootMeasurement.fmt(ttime)}"
    end
    puts "Total time spent requiring: #{BootMeasurement.fmt(@last - @first)}"
  end
end

alias kernel_base_require require
def require(file)
  BootMeasurement.start(file)
  kernel_base_require(file)
  BootMeasurement.finish(file)
end
{% endhighlight %}

Then in `boot.rb`, I require it:

{% highlight ruby %}
require File.join(File.dirname(__FILE__), '..', 'require_timer')
require 'rubygems'

# Set up gems listed in the Gemfile.
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)

require 'bundler/setup' if File.exists?(ENV['BUNDLE_GEMFILE'])
{% endhighlight %}

Then I run this simple runner script:

{% highlight ruby %}
BootMeasurement.report
puts "Done"
{% endhighlight %}

And I see something like 

{% highlight text %}
user@host:~/src/rails_app (develop) $ time bundle exec rails r booted.rb
( lots of lines elided... )
exception_notifier/notifier                    : 1.37 sec
yard-cucumber                                  : 1.63 sec
net/ssh                                        : 2.07 sec
net/ssh/gateway                                : 2.09 sec
capistrano/configuration/connections           : 2.17 sec
capistrano/configuration                       : 3.25 sec
/home/user/src/rails_app/config/environment.rb : 14.24 sec
/home/user/src/rails_app/config/application    : 29.57 sec
Total time spent requiring: 43.82 sec
Done

real 0m48.486s
user 0m46.247s
sys  0m1.344s
{% endhighlight %}

Not perfect, but gives me some insight!
