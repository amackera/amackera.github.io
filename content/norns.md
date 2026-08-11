+++
title = 'The Norns Series'
date = 2026-08-10T09:00:00-07:00
description = 'Every post about Norns, the durable execution runtime for AI agents, in reading order.'
+++

[Norns](https://github.com/nornscode/norns) is my open-source durable
execution runtime for AI agents, built in Elixir on the BEAM. If an
agent crashes mid-run, Norns replays its event log and picks up where
it left off. I've been writing about it since before it had a name.
The posts build on each other, so here they are in order.

1. **[Code That Cannot Fail](/posts/durable-execution/)** (Mar 2026).
   Why 911 calls and legal cases need durable execution, and the
   lineage from journaling file systems to Temporal.

2. **[Agents That Pick Up Where They Left Off](/posts/introducing-norns/)**
   (Mar 2026). Introducing Norns, and why the Erlang VM is the right
   substrate for agents that can't fail.

3. **[Kill the Worker, Keep the Run](/posts/norns-demo/)** (Apr 2026).
   A 30-second demo. The worker dies twice and the agent finishes the
   job anyway.

4. **[Mimir in Production](/posts/introducing-mimir/)** (Apr 2026).
   The first reference agent, a Slack bot with persistent memory, and
   the run that survived my laptop closing on a ferry.

5. **[Four Bugs From Production in Norns v0.3](/posts/norns-v0.3/)**
   (Jul 2026). Sub-agent context inheritance, and four bugs that
   running Mimir in production shook out of the runtime.

6. **[Norns Elixir SDK, v0.1](/posts/norns-elixir-sdk-v0.1/)** (Jul
   2026). The second official SDK, for building durable agents on the
   BEAM.

7. **[Is This Still Happening?](/posts/is-this-still-happening/)**
   (Aug 2026). Crash recovery was forking sub-agent work instead of
   reattaching to it. Fixed and shipped in v0.4.

There's also [Hermod](https://github.com/nornscode/hermod), a pull
request shepherd built on Norns that I haven't properly announced
yet.

If you'd rather run it than read about it, start with the
[hello agent](https://github.com/nornscode/norns-hello-agent) and the
[repo](https://github.com/nornscode/norns). Bug reports and issues
are very welcome.
