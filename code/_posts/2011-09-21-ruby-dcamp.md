---
layout: post
title: Ruby DCamp Retrospective
---

I attended [Ruby DCamp][1] this year. It's a programmers' event where 
you camp out (in cabins). There's a code retreat the first day and then
subsequent days are an [open-spaces][2] event.

### The Code Retreat

The code retreat involved programming [Conway's Game of Life][3] over
and over again. Each time you get 45 minutes to work on the problem. You
and another (pseudo-randomly selected) person pair program and use
[TDD][4]. At the end of the 45 minutes, you delete all the work you've
done. The goal is not necessarily to create a program that runs the
algorithm, but rather to hone your craft and focus on how you
pair-program and how you effectively use TDD. 

I ended up involved in only three of the total (six or so) sessions.
Each was a unique experience, and eye-opening. It made me remember why
we, as a community, care so much about TDD and other agile practices:
they really work.

### The Open-Spaces Sessions

Saturday and Sunday were self-organized sessions. The topics ranged from
specific (Programming with [Coffeescript][5] or using [DCI][6]) to broad
(Effective pairing practices, Intro to statistics).

Sunday had a number of hacking sessions where we worked on actually building
something. These were especially rewarding: I continued with the Game of
Life theme and built a version using [Backbone.js][7] and with [Erlang][8].

### New Concepts

The sessions around DCI, presenters, and fast-testing were probably the
most influential to how I'm planning to work in the future. I've been
moving towards the idea of breaking apart my thick models for awhile
(some of the models we work with at Optoro are over a thousand lines
long). These concepts give a strong conceptual framework to use while
refactoring our models down to something actually maintainable.

Thanks again to [@elight][9] and everyone who attended for making DCamp
such a great event!

[1]: http://rubydcamp.org/
[2]: http://en.wikipedia.org/wiki/Open_Space_Technology
[3]: http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
[4]: http://en.wikipedia.org/wiki/Test-driven_development
[5]: http://jashkenas.github.com/coffee-script/
[6]: http://en.wikipedia.org/wiki/Data,_Context_and_Interaction
[7]: https://gist.github.com/1225251
[8]: https://gist.github.com/1225511
[9]: http://www.twitter.com/elight
