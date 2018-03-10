---
layout: post
title: Crossing the no-downtime chasm - part 2 - Moving and breaking
---

Often the business knows before engineers do that the organization has passed the point of there being no acceptable amount of planned down time.

In this series of small posts I look at different things that get a lot harder to do when no amount of significant downtime is acceptable to the business.

[Part 1](http://blog.bittercoder.com/no-planned-downtime-chasm-part1/) [Part 2](http://blog.bittercoder.com/no-planned-downtime-chasm-part2/) [Part 3](http://blog.bittercoder.com/no-planned-downtime-chasm-part3/)

Moving data centers, or migrating to the cloud
----------------------------------------------

Though less of a problem for many startups these days - moving data centers, or migrating from a co-location or on-premise environment to a cloud provider is a massive undertaking.

Though it is definitely possible to do without down time - most businesses can be persuaded (at least for the first time they move) that some level of down-time can be accepted.

The question is... how much?  Again I think this is a function of time - the longer you leave the migration or move, the less tolerance for extended periods of down-time there will be.

*However* conversely I strongly believe that a reduced tolerance here is actually very beneficial - because it forces engineers to get to a position where they are able to automate a large amount of the migration process and rehearse prior to making the move.

In this one instance I strongly feel that the cost is not so much increased as just brought forward.  Automating the setup of your new environment will continue to yield benefits long into the future.

BUT if your focus is on opportunity cost - then delaying a move to the cloud is probably still a poor trade-off in most cases.  Experience and anecdotes from others in the industry normally indicates that by the time a move is completed, the team felt they could have benefited from the new infrastructure and capabilities 6-12 months earlier then the move, often that time leading up to the migration can result in some rather unsatisfactory trade-offs or technical debt accumulation.

There are conversely other things to think of in this case i.e. has the organization matured enough to operate in the new environment.

Lots of factors come into choosing the most appropriate time to commit to this kind of move - but generally it will be faster and easier to plan for an initial migration if your business has a higher tolerance for planned downtime to complete the migration.

Breaking out key parts of a system based around risk or compliance boundaries
-----------------------------------------------------------------------------

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