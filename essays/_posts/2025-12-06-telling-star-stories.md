---
layout: post
title: Telling STAR Stories
---

I was sharing with some friends that STAR stories are how recruiting and interviewing works at lots of places these days. Either they look for STAR directly, or they respond better to STAR-formatted storytelling. It also helps me write performance reviews too, so I get a lot of practice. 

Also, instead of "we" stories, interviewers want to know what you did yourself, not your team, even if it was in support of the team.

Here's an example of what that looks like, based on a real story from my time at Axios:

---

_(Situation)_ At Axios, when I was CTO, we were building our first mobile app. We'd chosen to enable our client to iterate independently from our backend servers, so we introduced a GraphQL layer so the client could recombine information as needed and still achieve efficient data usage. We'd been experimenting with new Elixir/Phoenix backends, but for this use-case we chose Apollo for our GraphQL server, as it was an industry standard and worked well for our team who already used and understood javascript in depth. We'd tested the backend system using simulated traffic, and everything looked good up to our normal peak web traffic - up to about 10k users simultaneously, showing just 1-2s of load time per page at peak traffic. We made the call and released the app, and started to market it in our daily newsletters. Pretty quickly though, our load times for articles shot up well beyond the 1-2s avg peak load times we'd tested - to 4-5s at P50, and 15s at P90!

_(Task)_ As CTO, I had to rally the team to resolve the problem and get page load times back down to at most our 2s tested peak article load time. I was under extreme time pressure, as Axios was still a fledgling news organization and our brand new app was being used by influential people who were starting to give us negative feedback. Time was of the essence and every minute counted!

_(Actions)_
I brought my team into a war room and we debated options. Some people wanted to try to improve how Apollo's cache was configured, others wanted to work on query performance and speed, others wanted to do more debugging to try to better understand the problems. I green-lit all those options, and allocated more people to caching because I felt that that was where we'd get the fastest improvements. My Director of Backend Engineering also suggested a moonshot: to have one of our new engineers—Julian, a relatively junior guy but super sharp—see if he could build a replacement for Apollo using Elixir/Phoenix. I decided to greenlight that too, because why not at this point, I didn't need him on the other tracks as much as the other engineers who were already there. I knew that Phoenix was a high-performance system, and it's why I'd had us investigating it in the past, so we could find innovative ways to improve our codebase. This felt like a great opportunity to "use it in anger."

Day slipped into day without improvements. I sat next to my engineers and tried to brainstorm and help as well. I tried writing caching layers on my own to see if I could take different approaches. We deployed probably a dozen times a day trying to resolve the Apollo speed problem, but to no avail. Pressure mounted and I was starting to get desperate. We didn't have much money for support, but I started to make the case to spend to purchase paid support from the Apollo company as well. 

I was preparing to execute a contract when my moonshot paid off! Julian grabbed me and asked to demo his solution. I anxiously joined him on Zoom, but he quickly blew me away—he'd built a full clone of the news article backend, in just 4 days, and in his simulated load testing it was FAST - just 250-500ms to load any given page. He'd relied heavily on Phoenix's ability to take on existing databases, and had pointed the Phoenix ORM at the current news database. He'd then enabled a Phoenix-native built-in GraphQL endpoint interface, which just Did The Right Thing and worked, including in-memory caching! It was incredible.

We huddled together as a team again and reviewed Julian's prototype. We tried to attack it with everything we could imagine, but it was bulletproof, so that Friday we shipped it and replaced the Apollo backend with this brand new fledgling Phoenix app. And it worked!

_(Result)_ That Friday, we took our P50 response times from up to 4 seconds down to 300ms, and our P90 from up to 15 seconds down to 500ms. And instead of seeing wild variance in response times, all clients received nearly identical performance for any given page.

Our customers were elated! The app they'd wanted since Axios launched was finally here, and its performance was rock steady and fast enough where speed was never a concern. We re-earned a lot of trust from our senior leadership team, and went on to build more features on the app over time.

---

So - plenty of "we" in that—especially in the situation and sharing the credit—but lots of "I" in the Task and Actions sections.

---

I also like to use simple STAR-formatted stories in my note-taking for people who report to me. I keep a personal file for everyone I work with, and when I notice something I want to keep around for performance-manageme time, I'll jot something simple down, like this:

S: building the new backend for mobile app; T: had to quickly prototype clone of GQL system in Phoenix; A: in 4 days, built a complete replacement that was FAST!; R: we replaced the Apollo backend and got repsonse times down to 300ms/500ms!!

---

I do this so much I even built a little vimscript for it:

```vim
function! STAREntry()
  call inputsave()
  let handle = input('Handle: ')
  let situation = input('Situation: ')
  let task = input('Task: ')
  let action = input('Action: ')
  let result = input('Result: ')
  let curtime = strftime('%c')
  call inputrestore()
  call setline('.', curtime . ' ' . handle . ' - Sit: ' . situation . ' - Task: ' . task . ' - Act: ' . action . ' - Res: ' . result)
  normal f-
  return ""
endfunction
inoremap <C-G> <C-R>=STAREntry()<CR>
```

Hope this helps you too!
