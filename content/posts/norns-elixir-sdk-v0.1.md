+++
title = 'Norns Elixir SDK, v0.1'
date = 2026-07-21T09:00:00-07:00
draft = false
+++

The [Elixir SDK for Norns](https://github.com/nornscode/norns-sdk-elixir/releases/tag/v0.1.0)
got its first tagged release today, Norns' second official SDK after
Python.

Elixir is dope, and Norns itself runs on the BEAM, so it's fitting
that agents written in Elixir now work great alongside it. v0.1 is
early but covers the core: a worker mode for registering agents and
tools, a client for sending messages and reading back run state, and
support for a few LLM providers beyond Anthropic.

```elixir
{:norns_sdk, "~> 0.1"}
```

A basic agent looks about like this:

```elixir
defmodule HelloAgent do
  use NornsSdk.Agent

  tool :say_hello do
    def run(%{"name" => name}), do: "Hello #{name}"
  end

  def config do
    %{
      name: "hello-bot",
      model: "claude-sonnet-4-20250514",
      system_prompt: "Greet the user by name.",
      tools: [:say_hello]
    }
  end
end

NornsSdk.Worker.start_link(agent: HelloAgent, url: "ws://localhost:4001")
```

That's all you need. Tools, agents, and start up the worker.

If you're on the BEAM, give it a try and [open an
issue](https://github.com/nornscode/norns-sdk-elixir/issues) with
whatever you run into.
