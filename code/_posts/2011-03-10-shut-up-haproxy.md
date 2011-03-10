---
layout: post
title: Shut Up, HAProxy!
---

We use [HAProxy][1] to load-balance requests across multiple Rails backends. It works great, but man does it clutter up the logs. Every N seconds (2 in our case), HAProxy requests our root url (DashboardsController#index) to ensure the site is up. This results in lots of log entries like this:

{% highlight text %}
Processing DashboardsController#index (for 10.0.0.205 at 2011-03-07 06:42:20) [GET]
  Parameters: {"action"=>"index", "controller"=>"dashboards"}
Redirected to http://www.example.com/session/new
Filter chain halted as [:login_required] rendered_or_redirected.
Completed in 1ms (DB: 0) | 302 Found [http://www.example.com/]
{% endhighlight %}

Every. Two. Seconds.

This made it really hard to read the logs, let alone `cap tail`. I finally got fed up and created this initializer:


{% highlight ruby %}
module ActionController
  class Base

    # Yup, these are class methods. ActionController is interesting---there's a
    #   class-level `process` and an instance-level one. They take different
    #   sets of arguments, too. We want to interrupt this at the class level,
    #   because that's where we have access to `logger` (via a cattr_accessor)
    class << self
      # Keep the old process method around
      alias orig_process process

      def process(request, response)
        if haproxy_request?(request)
          # I've never had to use this method before. 
          #   It's a nice solution, too bad I have to 
          #   construct a Proc for it.
          logger.silence do 
            orig_process(request, response)
          end
        else
          orig_process(request, response)
        end
      end

      def haproxy_request?(request)
        # Unlike most people, HAProxy doesn't send a User-Agent.
        request.parameters[:controller].to_s == 'dashboards' && request.parameters[:action] == 'index' && request.user_agent.blank?
      end
    end

  end
end
{% endhighlight %}

It works with Rails 2.3. I'm sure you can do it with Rails 3 somehow---in fact I'm betting it's easier in Rails 3---but I'm working with what I've got.

And now I can go back to reading the logs like a sane person.

[1]: http://haproxy.1wt.eu/
