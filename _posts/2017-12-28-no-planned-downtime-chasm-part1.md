---
layout: post
title: Crossing the no-downtime chasm - part 1 - Migrations
---

Often the business knows before engineers do that the organization has passed the point of there being no acceptable amount of planned down time.

It's not an event in a SaaS companies lifetime that's marked with a lot of fanfare - but as an engineer it's incredibly significant - you know when you've moved past it because suddenly things become a lot more complicated.   Things are HARD.  Lots of your skills to date are not transferable to this new world.

It's often not until you experience severe unexpected down time that the tolerance of the business for such down time is established/tested, often the last time you had significant unplanned down-time is the pre-cursor to the business making the decision (often at the board level) to no longer tolerate planned down time - even though planned and unplanned are not the same thing, to the business (and it's customers) the impact is much the same.

Lets look at some real-world examples of how decisions change once planned down time is not an option.

Data migrations
---------------

When using SQL based stores, as your database structure evolves you often need to reshape and move data around or backfill values into newly created columns.

When you are able to sustain short periods of down-time, the simplest way to do this is a from a set of SQL statement migrations applied to the whole table.

As you start to grow the volume of data in your application, some of those scripts are going start to taking longer and longer to run.

Eventually they will cause enough contention when run that it starts causing operations to time out on the live system.

While you are able to sustain some down time, you can move to running those scripts at off-peak hours.

But - once you move to no longer tolerating planned downtime - you are faced with moving to a back-fill job based approach - this normally entails:

* Creating a scheduled job.
* Having the job page/poll for new rows to transform.
* Writing code to perform the row transformations, log results etc.
* Ensuring adequate dwell between pages of items being updated or other expensive operations.
* Having a mechanism in-place for you to turn the job off immediately if it's causing issues.
* Making it resilient to redeploys.
* Wiring in some monitoring and metrics so you can track progress of the job.
* Feature flagging the job so it's initially enabled in your QA/Staging environments prior to production for testing and monitoring.
* Removing the job and all it's associated code once the backfill has been completed in production.
* Dealing with common problems like "thundering herd" issues when the process running the jobs is restarted, perhaps by randomizing the start time of jobs.

Contrasting that process to the relatively simple task of writing a one-off script... you can be looking at a task which would have originally taken a few hours but is now something which is measured in days or a couple of weeks depending on infrastructure.

Harder data migrations
----------------------

Backfills on tables normally assume that you can add new nullable/default columns for free, but then take your time to perform the backfill across all the existing rows.

Some migrations however are much more costly - for instance re-creating the clustered index for a table, changing the primary key type, or removing some form of ill-advised table inheritance design which seemed like a good idea at the time....

This can often happen when starting with a simple design and then not evolving or scaling it soon enough prior to crossing the no-downtime chasm - say for example using a shared 32bit integer key table across an entire database, rather then a 64 bit value, or not clustering a table frequently queried by customer on the composite of a customer key and unique ID.

When you are able to sustain a small amount of downtime this is normally OK, by migrating batches of tables to 64bit primary keys, then finally upgrading the keytable itself. 

Depending on the nature of data access strategy/ORM you may be able to do this with only small amounts of down time or degraded performance for the largest tables.

However once down time is not an option, suddenly things become incredibly hard.  You are now in the territory of having to devise a way to introduce the new design table alongside the old, often having to build up a suitable facade around both so that queries can be run against both tables with the results combined, and updates causing an in-place migration from the old to the new table, or by having some data unavailable until it has been moved into the new table.

Foreign keys can make this particularly hard to do and so you probably need to temporarily remove foreign keys entirely for the period of the migration.

Depending on your domain when faced with these challenges you may even begin considering how you can use that opportunity to break parts of the monolith apart into services.

Something that might take a couple of days of planning and execution is now a weeks or month long project.

You can easily loose person-weeks of time just discussing the problem in meetings and devising a strategy before you have even begun to execute on it.

In the [next part](http://blog.bittercoder.com/no-planned-downtime-chasm-part2/) we will talk about the impact no down time has on moving hosting or breaking up a solutions architecture.