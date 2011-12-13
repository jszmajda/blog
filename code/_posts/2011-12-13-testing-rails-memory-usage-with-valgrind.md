---
layout: post
title: Testing Rails Memory Usage With Valgrind
---

We've been having some trouble running out of memory on our production
servers ever since we upgraded our app to Rails 3.1 and Ruby 1.9.2
(p290). Servers will gradually use up all the memory on our (m1.large)
servers (7.5Gb of RAM) over the course of 24 hours or so.

Today one of our developers came across some topics ([1][1], [2][2]) on
StackOverflow, as well as a [post on HN][3] that mentioned it might be a
bug in ruby 1.9.2. I wanted to investigate that claim empirically, so I
needed to find a tool that could measure memory consumption.

After a little googling, I was reminded of [Valgrind][4], a suite of
tools for testing applications. Valgrind has an excellent memory usage
analyzer, [Memcheck][5].

I was having some trouble running it and honestly got a little
frustrated, but [Evan Weaver's blog post about testing ruby with
valgrind][6] at least gave me hope it was possible. I eventually
trolled through enough of the (excellent) [valgrind documentation][7] and
found that if I ran valgrind with `--trace-children=yes` that I could
get the full results from testing my rails app.

### Setup

I used version 3.7.0 of valgrind, which I built from source. Ubuntu also
ships version 3.6.1, which should work the same. You can simply run
`sudo apt-get install valgrind` to get it. Otherwise it's easy to
install from [source][8].

I'm using RVM to manage my rubies. Ruby supports valgrind internally as
of 1.9 with the --with-valgrind configure option. It's apparently on by
default, but if you're paranoid, you can pass the additional configure
flag with RVM by doing:

{% highlight text %}
rvm install 1.9.2 -C --with-valgrind
{% endhighlight %}

RVM's `-C` option will pass subsequent options to the configure script.

In order to more accurately test ruby, I wrote a test script to ensure
my application was exercised moderately. It's a simple script which just
loads some data into memory:

{% highlight ruby %}
Order.all.map{|o| o.customer }
{% endhighlight %}

To actually execute valgrind, I had to run:

{% highlight text %}
valgrind \
  --log-file=valgrind_output.log \
  --trace-children=yes \
  bundle exec rails runner test.rb
{% endhighlight %}

### Testing

Valgrind outputs a `HEAP SUMMARY` and a `LEAK SUMMARY` at the end of
program execution. These are what I'm focusing on to determine memory
changes between 1.9.2 and 1.9.3.

Here's the raw output:

**1.9.2-p290 Results:**

{% highlight text %}
HEAP SUMMARY:
    in use at exit: 187,561,529 bytes in 3,197,878 blocks
  total heap usage: 19,991,955 allocs, 16,794,077 frees, 6,663,565,008 bytes allocated

LEAK SUMMARY:
   definitely lost: 51,217,761 bytes in 543,305 blocks
   indirectly lost: 126,656,843 bytes in 2,508,233 blocks
     possibly lost: 3,542,851 bytes in 48,230 blocks
   still reachable: 6,144,074 bytes in 98,110 blocks
        suppressed: 0 bytes in 0 blocks
Rerun with --leak-check=full to see details of leaked memory

For counts of detected and suppressed errors, rerun with: -v
ERROR SUMMARY: 1857 errors from 21 contexts (suppressed: 5 from 5)
{% endhighlight %}

**1.9.3-p0 Results:**

{% highlight text %}
HEAP SUMMARY:
    in use at exit: 5,373,469 bytes in 83,901 blocks
  total heap usage: 19,385,645 allocs, 19,301,744 frees, 6,601,997,814 bytes allocated

LEAK SUMMARY:
   definitely lost: 6,536 bytes in 54 blocks
   indirectly lost: 5,899 bytes in 93 blocks
     possibly lost: 496 bytes in 3 blocks
   still reachable: 5,360,538 bytes in 83,751 blocks
        suppressed: 0 bytes in 0 blocks
Rerun with --leak-check=full to see details of leaked memory

For counts of detected and suppressed errors, rerun with: -v
ERROR SUMMARY: 2053 errors from 17 contexts (suppressed: 4 from 4)
{% endhighlight %}

### Analysis

As you can see, 1.9.2 leaked around 51M, whereas 1.9.3 leaked only
6.5k. In block terms, that's a **100,000x increase in leakage**.

The only thing that changed between these reports was the version of
ruby. It seems there is definitely a memory leak running at least our rails
3.1 app on ruby 1.9.2. We'll be upgrading to 1.9.3 as soon as possible.

Additionally, our app eats about 6.6Gb of memory performing this simple test.
I also ran a runner script with no ruby operations (just load the environment
and quit) and the app still consumed 4.3Gb. We definitely need to look
into what's going on to load all that data. For reference, here's the
abbreviated output of `rake stats`:

{% highlight text %}
+-------------+-------+-------+---------+---------+-----+-------+
| Name        | Lines |   LOC | Classes | Methods | M/C | LOC/M |
+-------------+-------+-------+---------+---------+-----+-------+
| Controllers | 11079 |  9065 |      89 |     855 |   9 |     8 |
| Helpers     |  1089 |   937 |       2 |     102 |  51 |     7 |
| Models      | 22278 | 14202 |     170 |    1609 |   9 |     6 |
| Libraries   | 17941 | 13447 |     183 |    1274 |   6 |     8 |
+-------------+-------+-------+---------+---------+-----+-------+
{% endhighlight %}

[1]: http://stackoverflow.com/questions/3204551/memory-leak-with-ruby-1-9-2-rails-3-0-beta-4
[2]: http://stackoverflow.com/questions/7582338/rails-3-1-memory-leak-for-missing-local-partial-variables-ruby-1-9-2-p290
[3]: http://news.ycombinator.com/item?id=3175890
[4]: http://www.valgrind.org
[5]: http://valgrind.org/info/tools.html#memcheck
[6]: http://blog.evanweaver.com/2008/02/05/valgrind-and-ruby/
[7]: http://valgrind.org/docs/manual/manual-core.html
[8]: http://valgrind.org/downloads/
