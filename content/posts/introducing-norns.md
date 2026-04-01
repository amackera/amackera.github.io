+++
title = 'Introducing Norns'
date = 2026-03-31T16:06:47-07:00
draft = false
+++

[Norns](https://github.com/amackera/norns) is a durable execution
runtime for AI agents. If an agent crashes mid-run, Norns replays its
event log and picks up where it left off. No lost state or duplicated
side effects. It's built in Elixir on the Erlang VM, and it's open
source.

In my [last post](/posts/durable-execution), I talked about why
durable execution matters when you're building software that handles
911 calls and legal cases. This post is about why I chose the
foundation I did.

## The Limits of Temporal

I love Temporal. I said so in the last post, and I meant it. At
LegalMate, durable execution saved us more times than I can count. But
as I started building agent infrastructure at Hyper, I kept running
into friction.

Temporal was designed for microservice orchestration: workflows
composed of activities, deterministic replay, known inputs and
outputs. An agent loop is a different shape. The number of steps isn't
known in advance. Tools come and go as workers connect and disconnect.
The execution graph isn't a DAG you define up front — it's an
open-ended loop driven by an LLM's decisions.

You *can* model this in Temporal, and people do. But you end up
writing a state machine inside a workflow, mapping LLM calls to
activities, managing conversational state yourself. The abstraction
was designed for a different problem, and the glue code makes you
feel it.

I wanted something purpose-built for this loop, on a runtime that
understood fault tolerance at a fundamental level.

## Why the BEAM

The Erlang VM (the BEAM) was built in the late '80s at Ericsson to
run telephone switches. The requirements were: handle massive
concurrency, never go down, recover from failures automatically, and
allow live upgrades without dropping calls. Sound familiar?

Replace "telephone switches" with "AI agents" and the overlap is
striking.

Most languages treat crashes as something to prevent. Forced to write
defensive code, you wrap everything in try/catch, and check every
return value. You build walls around any potential failure.

Erlang took the opposite approach: crashes are normal. Don't prevent
them. Let them happen. The system's job is to recover, not to avoid
failure. This isn't negligence, it's a design philosophy. Instead of
writing code that tries to handle every possible error state (and
inevitably misses some), you write code for the happy path and let the
runtime handle recovery.

On the BEAM, every unit of execution is a *process*, a lightweight,
isolated entity with its own memory. When a process crashes, it just
crashes. It doesn't corrupt shared memory or take down its
neighbors. A *supervisor* process watches it, notices the crash, and
decides what to do: restart it, escalate, or just let it go. This
happens automatically at the VM level. If you're building durable
agent execution in Go or Java, you end up reimplementing this,
building process managers, health checks, restart logic, and crash
isolation. The BEAM gives you all of that as the default behavior of
the runtime.

BEAM processes are not OS threads.  They're extraordinarily
lightweight (a few hundred bytes of initial memory) with isolated
heaps and per-process garbage collection.  You can run hundreds of
thousands of them on a single machine without even thinking about
it. The BEAM's scheduler preemptively multiplexes them across CPU
cores, so no single process can starve the others.  The concurrency
model isn't bolted on, it's built in.

This is the runtime Ericsson built for telephone switches that
couldn't go down. It's the right substrate for agents that cannot
fail. That's what Norns runs on.

## Norns

Norns is built for a specific class of agent: long-running,
tool-using agents with side effects and real failure-recovery needs.
The kind that sends emails, charges credit cards, dispatches
emergency services — where a dropped run or a duplicated action isn't
a minor inconvenience, it's a bug with consequences.

At it's core, Norns is a pure state machine. It never calls an LLM or
executes a tool. Its only job is to manage state transitions and
persist events. Every step of execution (LLM requests, responses, tool
calls, results, checkpoints) is written to an event log. If an agent
process dies, Norns replays from the last checkpoint and continues.
The agent doesn't know it was interrupted. It's the same principle as
Temporal, but native to the BEAM.

Under the hood, each agent is a `GenServer` (Erlang's abstraction for
a stateful, message-driven process) running under a
`DynamicSupervisor`. Process isolation, crash recovery, and massive
concurrency all come for free from the BEAM. What Norns adds is the
orchestration layer on top.

Concretely, imagine an agent handling an intake call. It transcribes
the caller's information, looks up the nearest dispatcher, and sends a
notification email. Then the container gets evicted. Norns replays the
event log, skips the completed steps, and resumes where it left
off. From the caller's perspective, nothing happened.

### Workers

All actual work happens on external *workers* that connect via
WebSocket. Workers register their tools and capabilities when they
connect. They hold all the API keys, Norns never sees them. They
execute LLM calls and tool invocations on behalf of the orchestrator.

If a worker container gets evicted mid-tool-execution, the
orchestrator notices the task didn't complete and puts it back in the
queue. The next available worker picks it up.

### Idempotency and Error Classification

When Norns replays an agent's event history after a crash, it can't just
re-execute every tool call. Some tools have side effects. You don't
want to send the same email twice or charge a credit card again
because a container bounced.

Side-effecting tools get deterministic idempotency keys, which are
derived from the run ID, step number, tool call ID, and tool name. On
replay, Norns checks for an existing result with the same key and
skips re-execution. The event log is the source of truth. If a result
exists, the tool already ran.

Error handling is equally deliberate. Not all failures are the same,
and Norns doesn't treat them the same. Every error gets classified:

- **Transient** failures (timeouts, flaky connections) get up to 3
  retries with exponential backoff. These are the "try again, it'll
  probably work" errors.
- **Rate limits** and **upstream unavailability** get patient retries
  -- up to 10 with linear backoff. The dependency is healthy, you
  just need to wait.
- **Validation** and **policy** errors are terminal. No retry. The
  input was wrong or the action wasn't allowed. Retrying won't fix
  it.

This matters because the default in most agent frameworks is to either
retry everything or retry nothing. Retrying a validation error wastes
time and money. Giving up on a transient network blip kills a
perfectly good run. The system needs to know what kind of failure it's
dealing with, and respond accordingly.

### Try It

Norns is early and the repo is public. There are SDKs for
[Python](https://github.com/amackera/norns-sdk-python) and
[Elixir](https://github.com/amackera/norns-sdk-elixir), and a
[hello world example](https://github.com/amackera/norns-hello-agent)
you can run in a few minutes. If you're building agents that need to
be reliable -- actually reliable, not "retry three times and log the
error" reliable -- try Norns on a toy project and
[open an issue](https://github.com/amackera/norns/issues) with what
you find.

## The Foundation

In my last post, I said that AI agents need the same durability
guarantees we've spent decades building for data. The BEAM is where
those guarantees live. Not as a library you import, but as the runtime
itself. Fault tolerance isn't a feature of Norns — it's a property of
the VM it runs on. Norns just makes it accessible for agents.

When someone dials 911, the agent that answers needs to be as reliable
as the telephone network it replaced. It's fitting that the technology
built to run those telephone networks is the right foundation for
what comes next.
