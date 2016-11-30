---
layout: post
title: A Surface Roadblock
---

This is a continuation of my previous posts, the last one being [More Surface Book]({% post_url 2016-11-28-more-surface-book %})

### Damnit Docker.

There is Docker for Windows, but there's no `docker` CLI in the WSL
environment. This might be a problem because we're shifting more and more to
using docker to deploy all our services. As the CTO I am occasionally required
to actually like, edit services and run them on my machine. So this might be a
big blocker. Certainly I can install an Ubuntu VM, but a big part of the point
of all this is to avoid having to use a VM. I'm going to think about this one a
bit, and even try the VM (with HyperV too, why not), but this might be the
biggest reason to stick with Apple for now.

Wow ok. Docker for Windows works by using Hyper-V. Hyper-V is Microsoft's
virtualization technology. When it's installed it locks out all other
virtualization technology, so like you can't run VirtualBox and Hyper-V
simultaneously and expect VB to get any kind of hardware virtualization
acceleration. So if I want to run a Ubuntu VM for dev work in the background
and still try to get Docker in Windows running then I have to have it in
Hyper-V.

That's probably not a terrible thing except that HyperV doesn't seem to have
some of the nice things I take for granted with VirtualBox, at least not
without more digging. Like a NAT'd IP for my VM so that I can always assume
it's at some internal IP. 

If I do run the VM I'll either have to use the (pretty crappy) virtual desktop
to get around the VIM ConEmu limitations or I'll have to deal with ConEmu
regardless. Starting to seem less and less worth it.
