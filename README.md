> **This repository has moved.** It now lives in [`ai-experiments/openai-agents-api`](https://github.com/techforum-repo/ai-experiments/tree/main/openai-agents-api), with its full commit history. This repo is archived and read-only.

# AI Agents Experimentation

Hands-on tests of OpenAI's **Agents API** (public beta, announced
2026-09-10) — the same harness that runs Codex and ChatGPT's agent mode,
exposed as an API primitive: an agent (model + instructions + tools) runs
inside a session, in either an OpenAI-hosted sandbox, a self-hosted
environment you connect yourself, or no environment at all.

Not a mock of what the docs describe — a real, runnable test against the
live API. Where the public docs skip the exact wire shape of something,
the code here follows the closest documented pattern and says so; verify
against your own account's beta access before trusting a result.

## [`openai-agents-api/`](./openai-agents-api)

Started as several separate probes (custom tool calls, sandbox code
execution, network allowlisting, package/workspace persistence, subagent
concurrency) that mostly worked as documented, but all shared one
recurring problem underneath: `openai_hosted` sessions often never
started their first turn at all — `idle` status, zero turns, forever,
despite the docs saying `input` at `sessions.create()` starts a turn
immediately.

That turned out to match a known OpenAI incident — ["Degraded
Performance affecting Agents API"](https://status.openai.com/incidents/01M2H3J1D6Y7RHAP49GRWGJAY0),
customers "unable to start turns in managed sessions" from 2026-09-14
13:30 PT, resolved 23:39 PT the same day — not a client-side
configuration problem. Every stuck session from the earlier probes was
run inside that window.

The consolidated test reruns the documented pattern on a real, verifiable
use case — clean and total a messy transactions CSV, fresh random amounts
every trial — across multiple live trials, checking the sandbox's answer
against an independently computed ground truth, not just self-reported
turn status.

**Verified live (2026-09-15): 5/5 trials started a turn and computed the
correct total**, turn-start latency ranging 3.5s–75.8s. Consistent with
the incident above having been the actual cause of the earlier failures,
not a client-side or config issue — see the folder's README for the
full per-trial breakdown.

## What this is not

A benchmark, or an endorsement. This is a beta API days old; names,
statuses, and event shapes here are read directly off OpenAI's docs (and,
where the docs were wrong, off the installed SDK's own generated types)
as of the date the test was written, and may have already moved by the
time you run this. Rerun before trusting any of this as durable — and if
turn start stalls again, check [status.openai.com](https://status.openai.com)
before assuming it's your configuration.
