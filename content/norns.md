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

{{< series "norns" >}}

There's also [Hermod](https://github.com/nornscode/hermod), a pull
request shepherd built on Norns that I haven't properly announced
yet.

If you'd rather run it than read about it, start with the
[hello agent](https://github.com/nornscode/norns-hello-agent) and the
[repo](https://github.com/nornscode/norns). Bug reports and issues
are very welcome.
