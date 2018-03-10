---
layout: post
title: Crossing the no-downtime chasm - part 3 - Finishing thoughts
---

Often the business knows before engineers do that the organization has passed the point of there being no acceptable amount of planned down time.

In this series of small posts I look at different things that get a lot harder to do when no amount of significant downtime is acceptable to the business.

[Part 1](http://blog.bittercoder.com/no-planned-downtime-chasm-part1/) [Part 2](http://blog.bittercoder.com/no-planned-downtime-chasm-part2/) [Part 3](http://blog.bittercoder.com/no-planned-downtime-chasm-part3/)

Migrating access tokens for 3rd party API's from individual accounts to shared operations accounts
--------------------------------------------------------------------------------------------------

Every other example I've listed so far in part 1 & 2 has been complex - but this is rather simple.

If you have any access tokens or API credentials issued to individual user accounts in use in production systems, then you should be aiming to eliminate those prior to crossing the no down-time chasm.

When a business is able to tolerate some down time this works out OK (not really... but shhh) - you can quickly respond, fix the account details and get back to business.

But once you have crossed the no-downtime chasm this becomes intolerable as inevitably when employees have their accounts disabled, key systems break and potentially you may encounter down time, or not be able to respond to some issue because part of your operation and alerting setup has stopped working.

There is little opportunity cost associated with fixing these kinds of issues before or after the shift to no down-time, so it's normally a no-brainer to do it before to eliminate risk.

Final thoughts
--------------

*Why did I write this series of posts?*

Well - mostly because I was wondering why we don't make a bigger deal about this?

I think we should - and I think as engineers we should be a lot more involved in the decision and scheduling of when to move to a process that no longer allows for scheduled down time.

As with so many things I think the reason we aren't involved is because we didn't preempt the businesses decision, by allowing the move to no longer tolerating planned maintenance being a business decision, and a reactive one at that, we rob ourselves of the control over making clear trade-offs around what technical debt or critical decisions we need to make prior to that move.

By preempting that decision I can see a lot of benefits:

* You can pay down technical debt or build out new features strategically based on cost prior to the move to cost afterwards when no planned down time is possible.
* Engineering and product can work together to ensure road-map features and projected growth are accounted for.
* You are able to celebrate and commemorate as a team the move to no longer tolerating planned down-time.   I think when engineers are not involved it can be quite jarring (it becomes another layer of additional stress, effectively eroding away at the sense of autonomy your role has).
* It means you can begin (if you are not already) recording and tracking suitable metrics for monitoring planned vs. unplanned down time on a weekly/monthly basis prior to the change in process, this gives the engineering team a way to communicate what has changed as a result.
* The process of moving ahead of the business or in tandem with the expectations of the business (and attaching a very real costs+benefits to the shift) pitches this as a "feature" or "capability" the business is gaining, rather then letting assumptions fill that void that it was something the engineering team should have been doing all along, but was not.

I have skirted around some nuances of this discussion (some SaaS platforms lend themselves to more graceful degradation then others in their design, even as a monolith) but even then there is likely always some part of the platform which the business is unwilling to tolerate planned downtime for such as identity/authorization.

There are also some arguments to be made for making the move to no planned downtime before getting your house in order or your engineers on board with the shift.

It does serve as a fairly brutal [*forcing function*](https://en.wikipedia.org/wiki/Behavior-shaping_constraint) for getting engineers to up-skill out of a classic monolith and arming them with some of the skills they will need to build out services in a distributed system - it is especially effective in establishing team knowledge in metrics, monitoring & alerts, processes like back-fills and learning how to strangle parts of the monolith so they can be replaced.   If you have the kind of engineering team that rises to the challenge of constrains this can work quite effectively.

That being said I always think the use of forcing functions should be carefully considered by engineering leadership - it's in essence a behavior-shaping mechanism that eliminates an element of freedom (autonomy) that can lead to engineer disenfranchisement or even generates the opposite outcome (you get the outcomes you asked for/upskill, but not necessarily the behavior you want or need in the long term (drive to self learn, identify and suggest new approachs to solving problems etc.) - and can possibly leave you scratching your head as to why engineers in your organization have not been growing into more senior IC and leadership positions over time).

But that's definitely a topic for another post!