---
title: "Code That Cannot Fail"
date: 2026-03-25
draft: false
---

At [Hyper](https://callhyper.com), we're building voice agents that
handle 911 calls. If our agent fails mid-conversation, a real human
being on the other end of the line isn't getting the help they
need. We can't afford to "just retry it." We need 911 to always work.

Before Hyper, I was at LegalMate, building automation for law
firms. We managed immigration applications, document discovery, data
migrations, client onboarding, and more. If a workflow failed
silently, if data was permanently lost, or an event didn't trigger, a
person's green card could be put at risk. A legal case can be won or
lost on a single document.

I've spent the last several years writing code that *cannot* fail.
Not "shouldn't fail" or "please hang up and try your call again." It
cannot fail. And that changes how you think about software
engineering.

So how do you make sure that software systems are rock solid in an
unreliable world?

## A Brief History of Not Losing Data

Let's trace the lineage of this old and important problem.

It started with **file systems and databases**. Journaling file
systems write intentions to a log before applying changes, so a
crash mid-write is recoverable. Database transactions formalized
this with ACID: a transaction either fully commits or fully rolls
back. The "D" in ACID stands for "durability."

**Replication** extended durability across machines. A single durable
database is great until the building floods. Distributed consensus
protocols like Paxos and Raft ensure data survives by maintaining
copies on multiple nodes.

**Event sourcing** flipped the model entirely. Instead of storing
current state, you store every event that led to it. The log *is*
the source of truth. If you have the full log, you can rebuild
anything from scratch.

## Enter Temporal

[Temporal](https://temporal.io) takes this entire lineage and applies
it to *code execution itself*. You write workflows as regular code,
and the Temporal server records every step. If the worker executing
your workflow crashes, another worker picks it up, replays the event
history, and reconstructs the exact state. Your workflow doesn't know
it was interrupted. It just keeps going.

It's event sourcing applied to program execution. The workflow's
event history is the commit log. Each activity completion is an event.
Replay rebuilds state. At LegalMate, I once watched a migration of
hundreds of gigabytes of legal documents blow up halfway through
because a special character in a filename hit an error deep in Windows
internals. Which files were migrated? Which ones were corrupted? With
durable execution, you don't have to ask. The workflow knows exactly
where it stopped, and it picks up from there.

## Durability in an Agentic World

So where does this leave us?

AI agents are moving off your laptop and into the cloud. They're
coordinating tasks, calling APIs, making decisions, and operating
autonomously for extended periods.

These agents run on infrastructure that fails in all the usual ways.
VMs get preempted. Containers get evicted. Networks partition. An
agent could be calling an LLM, waiting for human approval, or writing
to a database when the rug gets pulled out.

When someone dials 911, "the container got evicted" is not an
acceptable reason for the call to drop. You need the same durability
guarantees that we've spent decades building for data, but applied to
agentic execution.

At Hyper, this isn't theoretical. We're applying durable execution to
voice agents that handle real emergency calls. I've been building
something to make this easier, and I'll share it soon.
