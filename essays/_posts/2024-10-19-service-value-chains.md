---
layout: post
title: Service Value Chains
---

At AWS, we put a ton of work into tracking and improving the operational health
of our services. Every service has at least one operational dashboard, and most
have many sub-dashboards to enable operators to dive deep and resolve
operational challenges. Leaders at AWS review their services' dashboards deeply
and frequently to ensure the health of their services. Senior leaders at AWS
often review many services' dashboards, those that they own and those that they
don't, so that we can collectively raise the bar on the operational quality
that we deliver to customers. Operational quality is second only to security
here.

When I'm reviewing a service's operational dashboard, I'm particularly
interested in understanding that service's "value chain." This is the series of
steps or components that make up a customer's journey from problem through
continuous solution, and the ways you continuously measure that the service is
delivering on that value chain.

Here's an example: foo.com is an online store that sells camping gear. Their
customers go through the following rough steps when interacting with foo.com:

1. They determine they want to purchase some camping gear and find foo.com,
2. They interact with the site and find the equipment they want to purchase
3. They add the equipment to their cart
4. They complete the purchase flow
5. They receive shipment information and track their shipment
6. They may return items or interact with customer service
7. They leave reviews of the equipment.

In a service dashboard that's structured around the value chain of a service,
each of those steps is monitored (in varying ways), and all the dependencies
that support each step are monitored as well.

For example, for step 3, adding equipment to their cart, a service dashboard
might show a count of _add to cart_ interactions and their
[P99](https://stackoverflow.com/questions/12808934/what-is-p99-latency)
latency, along with perhaps metrics on CDN status (response latency), database
metrics (query latency), count of late-funnel pageviews by time (e.g. high
likelihood to add to cart), web server health metrics (green/yellow/red light,
latency), load balancer health metrics (tri-color light, latency), etc. . If a
dashboard page is showing multiple steps in the value chain then it could make
sense to group dependencies together, but I've found that repeating dependency
graphs across step dashboards can help operators put data in context and drill
quickly into problems with the context of the business in mind.

Monitoring service value chains helps operators understand the "why" behind the
graphs, and helps non-operators (product managers, etc) understand the "why"
behind the operations. This shared understanding helps develop a shared empathy
for the entire business among all people who contribute to it, and helps drive
not only better operations, but better business results. Not only that, but
operators can more effectively diagnose root causes of operational problems
because they can imagine the flow through the system more completely and put
into context the interactions that might've led to specific operational
outcomes. I've found that this context can make a big difference in fast
problem solving.

Also - I've seen that when you put the operations health of a service into
context like this you get better questions from reviewers. Folks are more
engaged in the review, we can more effectively learn and understand your
domain, and we can better help raise the bar on total outcomes for customers.
