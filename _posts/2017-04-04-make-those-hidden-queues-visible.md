---
layout: post
title: Make those hidden queues visible...
---

For a discipline so focused on measuring and understanding everything about our
software once it’s been carefully deployed into it’s final resting place in the
production environment — it always comes as a surprise when talking to engineers
about the process of reviewing, testing and deploying their software that it’s
so focused on “feelings” rather then facts.

Engineers can all tell you what “feels” bad about the process:

* Our automated acceptance tests fail often so we tend to ignore them
* Front end build takes forever
* Deployment to production is entirely manual
* Deployment to production has to be done by ops not devs
* Most of our time is spent on manual testing
* Manual deploy to production takes a long time because we need to stitch a bunch
of DB migration scripts together for each of the pull requests being shipped.
* (insert your own flavor of human misery)

Normally a little longer into the conversation you will end up having a heart
felt conversation where they might say something like:

> I keep telling the business we need to move to continuous delivery — but there’s
> no buy in from the business.

If you keep in touch with that person over the following year or so you might
even find the conversation evolves into:

> Yeah we have not adopted continuous delivery, but it wouldn’t really work for us
> it’s too risky because we need to be really careful about *<insert manual step
here>*

*You might identify with those quotes above, or be trapped in one of thse
ongoing conversations with a Friend at another company — So, what to do?*

**Stop hiding the pain**

Your build pipeline is a series of hidden queues. Make them visible.

Making hidden queues visible to me means everyone is aware of how long things
take — and robust discussions about those queues and their associated
costs/risks can take place in ways that a whole business can understand e.g
opportunity and cost.

Things you want to know about those queues:

* Does it in run in paralell with anything?
* Min/Max/Average/P90 times
* Is it getting faster/slower over time?
* Operational characteristics (how often do they succeed vs. fail or need to be
retried)
* Where do they need to run.

You might not realize it, but by keeping these queues hidden — your also hiding
the suffering of your engineering team at the same time.

**Tackle learned helplessness**

Bad things happen when people are suffering under the belief they are unable to
escape or even identify why they are suffering. For example when we hide the
reasons why things are painful…. well you see where I’m going with this I’m sure.

This suffering can often lead to a state of ‘[learned
helplessness](https://en.m.wikipedia.org/wiki/Learned_helplessness)’ — this
behavior modification is subconscious which lead to engineers holding strong
opinions (perversely) about not needing to change things.

Self-identifying you are suffering from learned helplessness can be difficult —
but if you are a new to an organization it’s often important to a conduct a
mental audit of who doesn’t believe things need to change and figuring out if
they are perhaps exhibiting symptoms of learned helplessness.

You will need to work out if you can route around those people, or better yet
work on making them an advocate for change (which can normally be achieved
through numbers and providing choices on how to improve things)

**Getting buy-in**

Businesses are ultimately simple creatures — they don’t want water, or blankets…
they just want your cash (or more specifically to make money, save money and
protect money).

For this reason I’m a big fan of [impact
mapping](https://www.impactmapping.org/) as a strategic planning technique — it
really helps with alignment of business, product and engineering .

When you are able to discuss product features in a framework the business
understands be it the impact on customers or average price point — it’s easy to
see how this can align with company goals around ACMR/ARPK etc.

But Impact mapping doesn’t have to be for Product — SRE activities such as build
pipelines fit really well into this framework as well.

Your pipeline once made visible will have a number of potential candidates for
improvement. Whatever you do, don’t tackle them all at once! Identify which
steps (if improved) will have the most impact to the thing you care about
(deployment cadence generally, but removing the risk of human interaction during
deployment is another) — counter these with the costs of these improvements
(call them internal features if you like - it can make it more palatable for
some businesses)— generally you will find that:

* Some wins will be easy, but might only buy you 30 seconds per build.
* Others might buy you 5 minutes buy are going to take an engineer away from
shipping features for a week.
* Some may be 1 month, but could shave 8–10 minutes off the build time, or provide
some resilience outcomes to the build pipeline.

Getting both the business and people opposed to any change into these
conversations (and at the same time) generally means instead of debating if you
should do anything at all, you move the conversation forward to “how heavily do
we want to invest” in making changes — the discussion about doing it has been
moved on to “what” not “if”.

Applying a little pricing psychology to these conversations can often work well
— Using [Anchoring ](https://en.wikipedia.org/wiki/Anchoring) by presenting an
expensive option alongside the option you think offers the business the best
bang for buck can make it easier to move the business toward the choice they
need to make.

On the same note — one of the worst things you can do is offer up 2 choices that
have the same impact/cost as this can lead to the business not wanting to choose
either option, differentiating the pipeline changes even a little can often make
it easier to get buy-in from the business.

Different organizations will have differing appetites initially, but regardless
of the outcome you have now started the business on the journey of improving
their build pipeline.

**Keeping momentum**

Once you have made your first improvements to build pipeline performance — you
need to make sure you do a few things:

* Surface that change back up to the business e.g. before / after measurements.
Celebrate these wins like you would shipping a product.
* Setup a process to deliver these measurements (times between shipping builds for
example, and how many times things are shipped, week by week) to engineering
leadership, with a graph of changes over time. Do this with at least a monthly
cadence.

Surfacing these details to engineering leadership provides insurance from two
closely related issues:

* Your pipeline degrading over time
* Lack of investment in the build pipeline given the growth in engineers (this is
particularly important to early/mid stage startups or those working in
hypergrowth startups)

Why? Because these are one of the few measurements that engineering team
leadership can surface to the business (that isn’t operational) which can show
that adequate investment is being made to accommodate for growth or churn in the
engineering team, or justify further investment.
