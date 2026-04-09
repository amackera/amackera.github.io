+++
title = 'Kill the Worker, Keep the Run'
date = 2026-04-08T17:27:31-07:00
description = 'A short demo of Norns recovering from worker crashes mid-execution'
draft = true
+++

I [introduced Norns](/posts/introducing-norns/) last week. Here's a
30-second demo of it actually working. I kill the worker twice
mid-run, and the agent finishes anyway.

<video controls width="100%">
  <source src="https://github.com/user-attachments/assets/b300b164-dc0c-44ea-a794-1de00b4f01a7" type="video/mp4">
</video>

## The Agent

The full example is in
[norns-hello-agent](https://github.com/amackera/norns-hello-agent).
The agent has one custom tool (`say_hello`) plus Norns' built-in
`wait` timer. The system prompt tells it to wait 10 seconds, then
greet the user by name. Simple on purpose.

Here's the worker:

```python
from norns import Norns, Agent, tool

norns = Norns("http://localhost:4001", api_key=os.environ["NORNS_API_KEY"])

@tool
def say_hello(name: str) -> str:
    """Greet someone by name."""
    return f"Hello {name}"

agent = Agent(
    name="hello-bot",
    model="claude-sonnet-4-20250514",
    system_prompt="You are a greeter. Your only job is to use the "
                  "say_hello tool, except you should use it after "
                  "calling the wait tool and waiting for 10 seconds. "
                  "After waiting 10 seconds, call the say_hello tool "
                  "with the person's name, if they provide it. "
                  "Otherwise, call them 'dude'.",
    tools=[say_hello],
    mode="conversation",
    on_failure="retry_last_step",
)

norns.run(agent, llm_api_key=os.environ["ANTHROPIC_API_KEY"])
```

And the client:

```python
from norns import NornsClient

client = NornsClient("http://localhost:4001", api_key=os.environ["NORNS_API_KEY"])
result = client.send_message("hello-bot", "Hello, I'm Anson.", wait=True, timeout=30)
print(f"Output: {result.output}")
```

`norns.run()` blocks forever and pulls tasks from the server -- same
pattern as a Temporal worker. The `wait` tool is built into Norns, so
the agent can call it without the worker defining it.

## The Demo

The video has three tmux panes: the client on the left, the worker on
the bottom, and `nornsctl runs tail 102` on the right streaming the
run's event log.

The client sends "Hello, I'm Anson." and the run starts. The worker
makes an LLM call, Claude responds with a call to `wait` (10
seconds), and I hit `Ctrl-C`. Worker dies. The event log shows
`llm_response` and `waiting_timer 10s`, then nothing.

I restart the worker. It reconnects, Norns replays the event log, and
execution picks up where it left off. It doesn't redo the LLM call --
that response is already in the log. It waits out the timer, gets the
result, makes another LLM call. Claude calls `say_hello`. "Hello
Anson."

I hit `Ctrl-C` again. Worker dies again. I restart it again. Norns
replays the full log, skips the steps that already completed, makes
one final LLM call, and the run finishes. The client prints:

```
Run 102: completed
Output: Hello Anson! Nice to meet you!
```

33 seconds total, two crashes included.

## Why This Works

Every step is persisted to an event log: `llm_response`, `tool_call`,
`tool_result`, `waiting_timer`, `completed`. You can see them
streaming in the `nornsctl` pane. When a worker reconnects, Norns
sends the full history. The worker rebuilds state from the log and
resumes from the last incomplete step.

Completed tools don't re-execute. Side-effecting tools get idempotency
keys derived from the run ID and step number -- if a result already
exists for that key, the tool is skipped. In this demo `say_hello` is
pure, but the same mechanism prevents duplicate emails or double
charges in production.

The client doesn't know any of this happened. It sent a message and
got a response.

## Try It

[norns-hello-agent](https://github.com/amackera/norns-hello-agent)
has everything you need. Clone it, start Norns with Docker, run the
worker and client, then kill the worker and watch it recover.
[Norns](https://github.com/amackera/norns) has the full source and
SDKs for Python and Elixir.
