---
layout: post
title: Crossing the no-downtime chasm - part 2 - Moving and breaking
---

Often the business knows before engineers do that the organization has passed the point of there being no acceptable amount of planned down time.

In this series of small posts I look at different things that get a lot harder to do when no amount of significant downtime is acceptable to the business.

Moving data centers, or migrating to the cloud
--------------------------------------------------------------------

Though less of a problem for many startups these days - moving data centers, or migrating from a co-location or on-premise environment to a cloud provider is a massive undertaking.

Though it is definitely possible to do without down time - most businesses can be persuaded (at least for the first time they move) that some level of down-time can be accepted.

The question is... how much?  Again I think this is a function of time - the longer you leave the migration or move, the less tolerance for extended periods of down-time there will be.

*However* conversely I strongly believe that a reduced tolerance here is actually very beneficial - because it forces engineers to get to a position where they are able to automate a large amount of the migration process and rehearse prior to making the move.

In this one instance I strongly feel that the cost is not so much increased as just brought forward.  Automating the setup of your new environment will continue to yield benefits long into the future.

BUT if your focus is on opportunity cost - then delaying a move to the cloud is probably still a poor trade-off in most cases.  Experience and anecdotes from others in the industry normally indicates that by the time a move is completed, the team felt they could have benefited from the new infrastructure and capabilities 6-12 months earlier then the move, often that time leading up to the migration can result in some rather unsatisfactory trade-offs or technical debt accumulation.

There are conversely other things to think of in this case i.e. has the organization matured enough to operate in the new environment.

Lots of factors come into choosing the most appropriate time to commit to this kind of move - but generally it will be faster and easier to plan for an initial migration if your business has a higher tolerance for planned downtime to complete the migration.

Breaking out key parts of a system based around risk or compliance boundaries
--------------------------------------------------------------------------------------------------------------------

Many startups building SaaS software these days start with a monolithic website, and maybe an API.   Then over time they quickly evolve to include other simple components like job scheduling, some kind of messaging/queuing layer to deal with asynchronous tasks and to allow back-pressure on restricted resources like 3rd party APIs, after which they start to diverge quite rapidly.

Normally this rapid period of growth is accompanied by a lot of flux in the business domain - engineering and product generally don't know exactly what it is they are building, or what the enduring primitives are that will make up their products DNA in the long term.

The pragmatic (maybe?) thing to do is to keep all these concepts closely together and jumbled, waiting for the opportunity to identify enduring concepts that fall within a bounded context suitable for extraction into their own smaller monolith or service.

Prior to crossing the no-downtime chasm, quickly refactoring and extracting out code and storage for these bounded contexts is quite easy.  You can take the system off line, copy database tables into new locations or even different storage technologies, remove foreign keys - handling this kind of migration is relatively simple, but carries with it a high degree of risk (it's often not easy to migrate back - so you get one shot).

However once over the chasm things get a lot more complex - suddenly you are faced with a task similar to rebuilding the engine on a plane while in flight - you will normally need to consider:

* Establishing ways to keep data synchronized between new and old systems, including monitoring how much latency there is in the replication process.
* Finding ways to feed read workload to the new service or implementation (and depending on risk that probably should only be a % of overall traffic) and monitoring.
* Depending on how drastically different the old vs. new systems are you might need to conduct scientist style experiments to confirm both implementations return similar results. https://github.com/github/scientist
* Building a back-fill process to publish all existing data into the new system.
* Switching write workload over to the new system.
* Deciding on an appropriate strategy e.g. almost like-for-like move first, then re-engineer after migration, or building the new system in the shape you want it to be moving forward, and then introduce a layer to adapt call sites to handle the old vs. new implementations of the system during the migration period, or some kind of hybrid.  Perhaps your new service is going to be fundamentally different (Event sourcing for example).
* Introducing mechanisms to provide additional resiliency around the network boundary you have introduced (automated retries, circuit breakers, throttling etc.)
* Security concerns at the network boundary.

On the flip side of this coin though is considerations like compliancy - PCI, HIPPA and others can quickly move from being small additional fixed costs per year to growing percentage costs as every part of your underlying system comes under scrutiny due to unclear boundaries.

The ability to wall off those compliancy considerations into a set of systems with a very small and isolated scope earlier in your companies growth curve can quickly yield benefits if you are facing spending person-years of engineering time focusing on satisfying auditing requirements.

Timing these kinds of moves depends an awful lot on the maturity of the organization and prior experience - no matter what - you will probably get it wrong on some dimension.

Best of luck!

Migrating access tokens for 3rd party API's from individual accounts to shared operations accounts
------------------------------------------------------------------------------------------------------------------------------------------------

Every other example I've listed so far has been complex - but this is rather simple.

If you have any access tokens or API credentials issued to individual user accounts in use in production systems, then you should be aiming to eliminate those prior to crossing the no down-time chasm.

When a business is able to tolerate some down time this works out OK (not really... but shhh) - you can quickly respond, fix the account details and get back to business.

But once you have crossed the no-downtime chasm this becomes intolerable as inevitably when employees have their accounts disabled, key systems break and potentially you may encounter down time, or not be able to respond to some issue because part of your operation and alerting setup has stopped working.

There is little opportunity cost associated with fixing these kinds of issues before or after the shift to no down-time, so it's normally a no-brainer to do it before to eliminate risk.



Final thoughts
---------------------

*Why did I write this post?*

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

That being said I always think the use of forcing functions should be carefully considered by engineering leadership - it's in essence a behavior-shaping mechanism that eliminates an element of freedom (autonomy) that can lead to engineer disenfranchisement or even generates the opposite outcome (you get the outcomes you asked for/upskill, but not necessarily the behavior you  want or need in the long term (drive to self learn, identify and suggest new approachs to solving problems etc.) - and can possibly leave you scratching your head as to why engineers in your organization have not been growing into more senior IC and leadership positions over time).

But that's definitely a topic for another post!