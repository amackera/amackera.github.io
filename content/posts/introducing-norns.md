+++
title = 'Introducing Norns'
date = 2026-03-31T16:06:47-07:00
draft = true
+++

In my [last post](/posts/code-that-cannot-fail), I talked about durable
execution and why it matters when you're building software that handles
911 calls and legal cases. I promised I was building something to make
durable execution easier for AI agents. This is that post.

But before I show you what I built, I want to explain *where* I built
it. Because the choice of runtime matters more than any feature list.
The substrate shapes everything above it.

## The Limits of Temporal

I love Temporal. I said so in the last post, and I mean it. At
LegalMate, durable execution saved us more times than I can count. But
as I started building agent infrastructure at Hyper, I kept running
into friction.

Temporal was designed for microservice orchestration. Its model is
workflows composed of activities: discrete, well-defined units of work
with known inputs and outputs. The workflow function is deterministic.
Activities are the side effects. Replay reconstructs state by replaying
activity completions.

The agent loop doesn't fit this shape. An agent's execution is
conversational. You send a message to an LLM, it decides to call some
tools, those tools produce results, the results go back to the LLM,
and it decides what to do next. The number of steps isn't known in
advance. Tools come and go as workers connect and disconnect. Sometimes
the agent needs to pause and wait for a human. The execution graph
isn't a DAG you define up front -- it's an open-ended loop driven by
an LLM's decisions.

You *can* model this in Temporal. People do, and it works. But you
end up writing a state machine inside a workflow, mapping every LLM
call and tool invocation to an activity, managing conversational state
yourself. You're paying the complexity cost of Temporal without
getting the ergonomic benefit. It's not that Temporal can't do it --
it's that the abstraction was designed for a different shape of
problem, and you feel that in every line of glue code.

I needed something built from scratch for this loop, on a runtime that
understood fault tolerance at a fundamental level.

## Why the BEAM

The Erlang VM -- the BEAM -- was built in the late '80s at Ericsson to
run telephone switches. The requirements were: handle massive
concurrency, never go down, recover from failures automatically, and
allow live upgrades without dropping calls. Sound familiar?

Replace "telephone switches" with "AI agents" and the requirements are
identical. Here's why.

### Let It Crash

Most languages treat crashes as something to prevent. You write
defensive code. You wrap everything in try/catch. You check every
return value. You build walls around failure and hope they hold.

Erlang took the opposite approach: crashes are normal. Don't prevent
them. Let them happen. The system's job is to recover, not to avoid
failure. This isn't negligence -- it's a design philosophy. Instead of
writing code that tries to handle every possible error state (and
inevitably misses some), you write code for the happy path and let the
runtime handle recovery.

On the BEAM, every unit of execution is a *process* -- a lightweight,
isolated entity with its own memory. When a process crashes, it just
crashes. It doesn't corrupt shared memory. It doesn't take down its
neighbors. A *supervisor* process watches it, notices the crash, and
decides what to do: restart it, escalate, or let it go. This happens
automatically, structurally, at the VM level.

If you're building durable agent execution in Go or Java, you end up
reimplementing this. You build process managers, health checks,
restart logic, crash isolation -- all the things the BEAM gives you
for free. You can get there, but you're fighting your runtime the
whole way.

### Process Isolation

Each agent on Norns is its own process. A GenServer -- Erlang's
abstraction for a stateful, message-driven process -- running under a
DynamicSupervisor. One agent's bad day doesn't cascade. If an agent
encounters an unrecoverable error, that process dies and the
supervisor handles it. Every other agent keeps running, unaware
anything happened.

But it goes further. In Norns, the orchestrator is a pure state
machine. It never calls an LLM. It never executes a tool. All of that
happens on external *workers* that connect via WebSocket. The
orchestrator manages state transitions and persists events. The
workers do the dangerous stuff -- the network calls, the API
integrations, the code execution.

If a worker container gets evicted while running a tool, the
orchestrator doesn't care. It notices the task didn't complete, and
reschedules it. The next available worker picks it up. The agent's
state is untouched because the agent's state was never at risk.

### Massive Concurrency

BEAM processes are not OS threads. They're not even goroutines.
They're extraordinarily lightweight -- a few hundred bytes of initial
memory -- with isolated heaps and per-process garbage collection.
You can run hundreds of thousands of them on a single machine without
thinking about it.

Each agent conversation is a process. Each WebSocket connection is a
process. The supervisor tree, the registry, the task queue -- all
processes. The BEAM's scheduler preemptively multiplexes them across
CPU cores with reduction counting, so no single process can starve
the others.

This means you can have thousands of concurrent agent conversations,
each maintaining its own state, each independently supervised, without
any shared-memory coordination. The concurrency model isn't something
you bolt on. It's the foundation.

And the BEAM gives you more: hot code reloading so you can deploy
without dropping running agents, distribution primitives for
clustering across nodes, and thirty years of battle-testing in telecom
systems that could not go down. Erlang solved this class of problem
decades ago. Building a durable agent runtime anywhere else means
rebuilding what already exists here.

## Norns

[Norns](https://github.com/amackera/norns) is a durable runtime for
AI agents on the BEAM.

### The Orchestrator

The core of Norns is a pure state machine. It never calls an LLM. It
never executes a tool. Each agent is a GenServer process under OTP
supervision, and its only job is to manage state transitions and
persist events.

Every step of execution -- LLM requests, responses, tool calls,
results, checkpoints -- is written to an event log. If an agent
process dies, Norns replays from the last checkpoint and continues.
The agent doesn't know it was interrupted. Same principle as Temporal,
native to the BEAM.

### Workers

All actual work happens on external workers that connect via
WebSocket. Workers register their tools and capabilities when they
join. They hold all the API keys -- Norns never sees them. They
execute LLM calls and tool invocations on behalf of the orchestrator.

If a worker container gets evicted mid-tool-execution, the
orchestrator notices the task didn't complete and puts it back in the
queue. The next available worker picks it up. The agent's state is
untouched because the agent's state was never at risk.

### Idempotency and Error Classification

This is where it gets interesting. When Norns replays an agent's event
history after a crash, it can't just re-execute every tool call. Some
tools have side effects. You don't want to send the same email twice
or charge a credit card again because a container bounced.

Side-effecting tools get deterministic idempotency keys -- derived
from the run ID, step number, tool call ID, and tool name. On replay,
Norns checks for an existing result with the same key and skips
re-execution. The event log is the source of truth. If a result
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
itself. Fault tolerance isn't a feature of Norns -- it's a property
of the VM it runs on. Norns just makes it accessible for agents.

When someone dials 911, the agent that answers needs to be as reliable
as the telephone network it replaced. It's fitting that the technology
built to run those telephone networks is the right foundation for
what comes next.
