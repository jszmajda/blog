---
layout: post
title: Surface Book
---

I'm trying out the new Surface Book with the performance base as my main
computer. It's been.. interesting! Here are some ramblings about my experience:


## Initial thoughts

I installed a bunch of stuff. Primarily [Windows Subsystem for Linux][1], but
also normal stuff (Chrome, etc), and the nicest terminal emulator I could find,
[ConEmu][2]

It's definitely different! It took me a good 4-5 hours to just remap all the text
movement keys in my head. I had gotten pretty used to using the mac key and
ctrl-e ctrl-a etc.

Apple has a tightly locked-down OS but they have spent time making some things
really really nice. Like plugging into an external monitor - the machine
remembers what my display configuration was for that setup and returns there,
so I can go big text on my mac when I'm plugged in. Windows doesn't do that but
it's manageable.

Windows text rendering is _much_ crisper. I remember now moving to the mac and
thinking everything was fuzzytown. Really nice to feel crisp!

Missing the cmd-{ and cmd-} keyboard shortcuts from apple. Ctrl-tab is ok but
more awkward for my fingers. It helps to have remapped caps-lock to ctrl though
(weird that that's a regedit thing).

## Black Friday

I didn't use it much yesterday, with turkey and all.

I'm starting to get used to it now, it's feeling more comfortable. Still mildly
annoying in how the keyboard feels (mostly just different). I've been tweaking
some of the customization stuff a bit more. I think I have the resolution
(really font scaling - odd but effective) stuff where I like it now. It
defaults to 200% and I'm using 175%. Things are a _little_ small but
manageable.

The trackpad is.. well it's good for a windows computer but that's not saying
much at all. Apple really dominates trackpad hardware. If the 3 year old MBP I
was using's trackpad is 100%, the newly released larger one is like 110% and
this surface's trackpad is like 40-50%. Most laptops would clock in at around
10%. Really no comparison.

The keyboard is a little mushy but again pretty good for a windows machine. I'm
picky about keyboards and I think I can live with it.

The screen is very nice, huge and high resolution, bright and crisp, great
color reproduction.

The machine is pretty snappy, but that's just based on UI feedback so far.
Haven't tried to like, compile something or whatever yet.

Battery life has been fine (haven't really paid attention, but I've been mostly
unplugged all day and it's at like half)

The trackpad has the option to right click by clicking on the bottom-right of
the trackpad (like where a button would be on a lot of other trackpads). You
can enable/disable it in the settings. I initially disabled it because I was
used to the mac not forcing me to pick, but I've reenabled it because there's
no option-click alternative on windows. I'll see how maddening it is, mostly
it's just a little weird.

Speaking of the touchpad and gestures - I _really_ miss the three-finger-drag
gesture the mac has. The Surface supports most of the gestures I used to use on
the mac (desktop swiping, window revealing (expose), app switching), but that
three finger drag.. just so useful. Best I can do with windows is double-tap
and drag, which is pretty meh by comparison.  Also the gesture recognition is
_slow_! The mac got it right away where this thing has a tiny but noticeable
lag. So frustrate.

One of my big still-unproven hypotheses is that I'll be happier with this
machine because I can tweak it more. Win10 is more tweakable to a point I think
but I'm not yet sure where the limits are of my needs and its ability to
support them. TBD!

Relearning all these text motion shortcuts is a bear.. ctrl-shift vs
shift-home, etc. I had gotten used to cmd-gestures with the mac.

The whole detachable tablet thing is kinda nice. I've been detaching it and
reattaching it in a variety of configurations during a variety of use-cases and
it has been nice. More of a small sweetener IMO though still. I doubt I'll want
to use it as a clipboard for taking notes, for example, and I already rarely
use a tablet at home for reading, so having the keyboard attached doesn't
really bother me. The on-screen input systems win10 has are.. well.. alright I
guess. I much prefer Android systems for all that. Win10 has a handwriting
recognition input mode.. it's honestly pretty mediocre. I used handwriting
recognition on my Compaq iPaq back in like 2000 and it was almost as good as
this.

Oh and I do much prefer Windows's font rendering to OSX's. _Much_ crisper,
makes me happy :)

FWIW I'm writing this using neovim in the Windows Subsystem for Linux
environment. It all works fine and I have a decently nice terminal app
(ConEmu).

HUH well I did just find that neovim doesn't handle arrow-key input quite
correctly so it's back to standard vim for me for now.

### Ruby

I tried rbenv because the macheads like it and I was starting to switch to it
on my mac. Turns out it wouldn't install 2.3.3. It doesn't do that really nice
`rvm requirements` thing that rvm does. So back to rvm. RVM is also so much
more verbose and I like that. Don't just sit there silently while you do god
knows what, please. 

Yep ok RVM installed 2.3.3 just fine so goodbye rbenv

Vim - the magic buffer `"*` isn't supported. That buffer gives you access to
the system clipboard. Sad. I use it a lot to copy/paste things in and out from
vim. That alone _might_ be a dealbreaker.

### More non-ruby

This face unlock thing is pretty sweet actually. I hate typing my password all
the time. I wonder how secure it actually is.

Interesting, plugging in a mouse - the mouse scrollwheel goes in "classic" (not
natural) mode, whereas the trackpad is in natural mode by default. /shrug

Man it does feel good to use apt-get again.. Homebrew is nice and all but
apt-get is life.

## More

More at my next post, [More Surface Book]({% post_url 2016-11-28-more-surface-book %})

[1]: https://msdn.microsoft.com/en-us/commandline/wsl/install_guide
[2]: https://conemu.github.io/
