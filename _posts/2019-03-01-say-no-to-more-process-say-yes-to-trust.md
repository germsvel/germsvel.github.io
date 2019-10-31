---
layout: post
title: "Say no to more process, say yes to trust"
date: 2019-03-01
categories: blog
tags: process, workflow
excerpt: >
  Adding policies is easy. Removing them is difficult. Say no to more process.
  Trust your team instead.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/say-no-to-more-process-say-yes-to-trust">thoughtbot.com/blog/say-no-to-more-process-say-yes-to-trust</a>.
</div>

When creating products, saying _no_ to things is vital. We must say no to things
because there are always too many features to build, too many directions we can
take, and too many policies we can adopt. But it is also extremely hard to say
no. So I am here to encourage you to say no a lot more.

As developers, designers, and product managers, one area where we should say no
frequently is when someone wants to introduce a new policy to our process in
response to an issue. Oftentimes, that issue will be surfaced during a
[retrospective], but it could come about in other ways.

And the issue is likely very real. I am not here trying to minimize that. We
should address the issue. But too often we default to introducing a new policy
into our process because (a) it shields us from blame for future mistakes (e.g.
"it was the process that failed, not me"), and (b) it makes us _feel_ safe from
future mistakes (e.g. "issue x cannot happen again because of policy z"). But I
think that 9 out of 10 times the better solution is to clearly communicate
problems and possible solutions and to trust your team to implement them.

Let me give you a couple of examples that might make things more concrete.

[Rework]: https://basecamp.com/books/rework
[retrospective]: https://thoughtbot.com/playbook/planning/meet-weekly-to-discuss-successes-failures-and-plans

## Scenario 1: The bug that raises the gates

Suppose a bug was introduced into your product. The bug was caught, and the team
fixed it but not before some clients complained. During a retrospective the
issue is brought up and someone suggests, "this _should_ have been caught in
code review. How come no one caught this? From now on Jeff (a particularly
talented developer/designer) must approve all code reviews. And let's set the
GitHub requirement that two people must approve the pull request before any code
can be merged."

In this example, the issue is very real: a bug was introduced that affected
clients. The solution, however, seems like an overreaction. The team should
certainly do code reviews, but we have suddenly turned Jeff into a gatekeeper.
Now pull requests will likely be backed up because Jeff, a single person with
limited time, must review and approve every single one of them. To make matters
worse, we have shifted the responsibility for the code being introduced away
from the individual with the most intricate knowledge of the feature (the one
opening the pull-request) to someone with less knowledge (in our case, Jeff).

### What's the alternative to gatekeeping?

I am not saying that having multiple people review your code is bad, especially
if one of them is a very talented developer/designer in your team. I think it's
great. But there is an alternative to introducing that policy. The alternative
is to trust that you can speak to your team about the issue and that they will
behave responsibly in fixing it.

Back to our example: I think it would be better to say "Team, let's be more
proactive in code reviewing. Don't merge your pull requests unless you've gotten
good reviews and feedback. If you haven't gotten code reviews, don't be shy. Ask
for more code reviews." This empowers developers and designers to be the owners
of what they are introducing into the code base, and it does so by trusting
them. Moreover, as the experts in the feature they are introducing, they may
know who would be an essential reviewer for their feature, and they can ask that
person accordingly. So trust that your team is comprised of responsible adults
who can behave professionally.

At thoughtbot, we have two points in our [code-review guides] that highlight
that trust and responsibility,

* Remember you (the reviewer) are here to provide feedback, not to be a
  gatekeeper.
* Final editorial control rests with the pull request author.

[code-review guides]: https://github.com/thoughtbot/guides/tree/master/code-review

## Scenario 2: The unspecified tickets and the day-long meeting

Suppose that during the last iteration two tickets were not specified well. As a
result, it took lots of back-and-forth conversations between a developer and the
product manager in order to figure out what needed to be done. During the
retrospective the developer brings this up, and the team decides to implement an
all-team meeting every Monday to go through every single ticket for the upcoming
iteration.

Once again, the issue is very real. When tickets are underspecified, and a lot
of work needs to be done before they are even ready for implementation, it can
cause serious delays in shipping features. But we have now turned a situation
that cost a developer some time into a weekly meeting that takes 1 hour for 8
people. That's 8 hours combined, a full day of work!

### What's the alternative to underspecification?

As with the previous example, the alternative I would suggest is communication
and trust. Inform the product manager that the tickets were underspecified. Ask
if there was something that blocked them when they were writing it. In this
instance, it may be that the product manager needed to ask for more input from
developers and designers before the iteration started. But instead of creating a
mandated meeting, trust that the product manager will be more proactive in
getting input when they feel is needed. If the problem resurfaces, revisit it at
the next retrospective. There may be a different blocker next time.

## Adding policies is easy. Removing them is difficult.

Introducing new policies tends to be easy. But removing outdated policies can be
very difficult. And the more policies you have, the more rigid your process
becomes. That rigidity will slow down the rate at which your team can iterate.
So even though the cost of introducing a single policy may seem small, the costs
compound quickly.

I think a quote from [Rework] in the chapter **Don't scar on the first cut**
really captures the essence of the problem,

> Policies are organizational scar tissue. They are codified overreactions to
> situations that are unlikely to happen again.

The alternative of communication and trust allows the team to remain flexible
and iterate quickly. It also empowers co-workers by trusting that they will
responsibly address the issue without the need for a mandated decree. Only
add policies to your process when it is absolutely necessary. Don't scar on the
first cut.

## What's the right amount of process?

It is true that the right amount of process will vary in each company, and the
process must adapt as teams grow and change. At thoughtbot we have found great
success with a [lightweight process] and two core concepts that encourage open
communication and trust, daily [standups] and weekly [retrospectives]. And we
have found that, often enough, that is enough.

[lightweight process]: https://thoughtbot.com/playbook/planning/manage-priorities-with-a-lightweight-process
[standups]: https://thoughtbot.com/playbook/planning/daily-standups-build-trust
[retrospectives]: https://thoughtbot.com/playbook/planning/meet-weekly-to-discuss-successes-failures-and-plans
[running a retrospective]: https://thoughtbot.com/upcase/videos/running-a-retrospective
