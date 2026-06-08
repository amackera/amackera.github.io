---
title: "A Busy Spring"
date: 2026-06-08
draft: false
---

The last few months have been busy. A lot happened, so I wanted to
write some of it down.

## Hyper × Motorola Solutions

[Hyper](https://callhyper.com), my employer, was acquired by
[Motorola Solutions](https://www.motorolasolutions.com). This has
been in the works for a long time, and I'm finally able to talk
about it (and found the time to write about it). It's a great
outcome for the company and I'm excited for what the future has in store.

Hyper is a natural fit. Motorola already has deep roots in public
safety: Computer Aided Dispatch, call handling, radios, and a whole
plethora of first responder tech. Adding AI-powered voice agents to
that stack makes a lot of sense. It's an opportunity to scale what
we've built to a much broader audience. Real impact on real people.

The transition has been interesting. I'm learning how
enterprise-scale organizations operate. The size and complexity of
a company like Motorola could easily grind things to a halt, but
things still get done. Decisions get made, products ship, people
find ways to move things forward. Hyper has a good home there.

## Hermod

On the side, I've been working on something I'm calling
**[Hermod](https://github.com/nornscode/hermod)**, a pull request
"shepherd." In the age of AI-assisted development, writing code is
getting easier every day. It's no longer about producing the code,
it's about managing the queue of pull requests that need review,
revision, and merging. Hermod is my attempt to help folks stay on top of that.
I'm not ready to announce it properly yet, it's still very much in
progress. But I wanted to make a note of it here.

Hermod is also another reference implementation for
[Norns](https://github.com/amackera/norns), my durable execution
runtime, and a great example of why durable agents are so hard.
Pull requests aren't
quick tasks. They can last days, weeks, or even months. An agent
that shepherds PRs through their lifecycle needs to keep context
across long time horizons, checkpoint its progress, and be resilient
to failure. That's what makes the shepherd reliable and trusty, and
it's a hard thing to build well.

More on this soon.
