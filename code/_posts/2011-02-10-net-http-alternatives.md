---
layout: post
title: Net::HTTP Alternatives
---

Here at [Optoro](http://www.optoro.com) we've been building some external tools that interact with our website via http. For simplicity's sake, we've been using [curl][1].

I was discussing this with some friends when someone recommended I check out [Curb][2]. I was already in the middle of transitioning to [HTTParty][3], due to its improved syntax, but I figured I'd check them all out in a head-to-head HTTP deathmatch.

I wrote this simple [Sinatra][4] app to hit for testing purposes:

{% highlight ruby %}
require 'rubygems'
require 'sinatra'

get '/' do
  "Hi"
end

post '/' do
  "HiPost"
end
{% endhighlight %}

And then a [benchmark suite](https://gist.github.com/821485 "Benchmark Code Gist"):

  <div class="shortCode">

{% highlight ruby %}
require 'rubygems'
require 'httparty'
require 'curb'
require 'net/http'
require 'benchmark'
include Benchmark

RUNS = 1000

url = 'http://localhost:4567/'
puts "Measuring GETs"
Benchmark.benchmark(" "*10 + CAPTION, 10, FMTSTR) do |x|
  x.report("Net::HTTP") do 
    RUNS.times do 
      # NOTE: This works slightly faster without URI.parse, but I'm trying to make this a little more real-world.
      Net::HTTP.get URI.parse(url)
    end
  end

  x.report("HTTParty") do
    RUNS.times do
      HTTParty.get(url)
    end
  end

  x.report("Curb") do
    RUNS.times do
      Curl::Easy.perform(url)
    end
  end

  # NOTE: This situation might not be typical, but it is a lot faster.
  x.report("Curb-reuse") do
    c = Curl::Easy.new
    RUNS.times do
      c.url = url
      c.perform
    end
  end

  x.report("curl") do
    RUNS.times do
      `curl "#{url}" 2>/dev/null`
    end
  end
end


params = {'q' => 'test'}
puts "Measuring POSTs"
Benchmark.benchmark(" "*10 + CAPTION, 10, FMTSTR) do |x|
  x.report("Net::HTTP") do 
    RUNS.times do 
      Net::HTTP.post_form URI.parse(url), params
    end
  end

  x.report("HTTParty") do
    RUNS.times do
      HTTParty.post(url, :body => params)
    end
  end

  x.report("Curb") do
    curb_params = params.collect{|(k,v)| Curl::PostField.content(k, v) }
    RUNS.times do
      Curl::Easy.http_post(url, curb_params)
    end
  end

  x.report("Curb-reuse") do
    c = Curl::Easy.new
    curb_params = params.collect{|(k,v)| Curl::PostField.content(k, v) }
    RUNS.times do
      c.url = url
      c.http_post curb_params
    end
  end

  x.report("curl") do
    curl_params = params.collect{|(k,v)| "-F \"#{k}=#{v}\"" }.join(" ")
    RUNS.times do
      `curl -X POST -s #{curl_params} "#{url}" 2>/dev/null`
    end
  end
end
{% endhighlight %}

  </div>

Here are the results:

<div class="highlight"><code><pre>
$ ruby test_http_clients.rb 

Measuring GETs
                user     system      total        real
Net::HTTP   0.520000   0.200000   0.720000 (  1.334052)
HTTParty    0.670000   0.250000   0.920000 (  1.616191)
Curb        0.330000   0.200000   0.530000 (  1.477893)
Curb-reuse  0.110000   0.040000   0.150000 (  0.830893)
curl        0.060000   0.980000   1.040000 (  8.760388)

Measuring POSTs
                user     system      total        real
Net::HTTP   0.730000   0.180000   0.910000 (  1.577040)
HTTParty    0.780000   0.240000   1.020000 (  1.677723)
Curb        0.370000   0.150000   0.520000 (  1.425708)
Curb-reuse  0.160000   0.030000   0.190000 (  0.849151)
curl        0.110000   1.130000   1.240000 (1202.826407)

</pre></code></div>

Clearly, libcurl-based [Curb][2] is the fastest alternative. It's not all that much faster than (pure-ruby) [HTTParty][3], though, so I think the improved syntax of [HTTParty][3] will keep me on that. What's truly interesting though is how slow shelling-out to curl is. Even though the 'total' time is negligibly more, the 'real' time it takes to open that subprocesses and execute [curl][1] is far longer, 8.7 seconds total for plain GETs, and a whopping 1202 seconds for POSTs.

[1]: http://curl.haxx.se/
[2]: http://rubygems.org/gems/curb
[3]: http://httparty.rubyforge.org/
[4]: http://www.sinatrarb.com/
