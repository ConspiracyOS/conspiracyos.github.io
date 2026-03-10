---
layout: page
title: Documentation
permalink: /docs/
---

# Getting Started

## Prerequisites

- Docker or Podman
- Go 1.22+ (both paths require building a Go binary)
- An API key (Anthropic or OpenRouter)

The container runs Ubuntu 24.04 or Debian 12 with systemd as PID 1.
Your host just needs a container runtime.

## Quick Start

Build the `conos` CLI, then let it handle the rest.

```bash
git clone https://github.com/ConspiracyOS/conos.git
cd conos && go build -o conos ./cmd/conos/
```

`CONOS_API_KEY` is the env var that holds your LLM provider's API key (Anthropic, OpenRouter, etc.). The name is configurable via `api_key_env` in the config.

```bash
CONOS_API_KEY=sk-your-key-here ./conos install
```

This pulls the image, writes the env file, and starts the container.
One command, one running system.

```bash
./conos status
```

## Build from Source

For contributors, or if you want to modify the inner runtime.

```bash
git clone https://github.com/ConspiracyOS/conctl.git
git clone https://github.com/ConspiracyOS/container.git

# Build the inner binary
cd conctl && go build -o conctl ./cmd/conctl/ && cd ..

# Configure
cd container
cp .env.example srv/dev/container.env
# Edit srv/dev/container.env — set CONOS_API_KEY=your-key-here

# Build and deploy
make image CONCTL_BIN=../conctl/conctl
make deploy
```

The container boots, provisions agent users and inboxes, and starts
watching for tasks. Verify with `make status`.

## First Task

Talk to the concierge. It routes your request to the right agent.

With `conos`:

```bash
conos agent "What agents are running in this conspiracy?"
conos agent logs -f     # watch the audit log
```

With `make` (from the `container` repo):

```bash
make task MSG="What agents are running in this conspiracy?"
make logs               # tail the audit log
make responses          # check agent outboxes
```

The concierge reads the task from the outer inbox, picks the right
agent, and routes it. Responses land in agent outboxes.

To task a specific agent directly:

```bash
conos agent task sysadmin "Run the contracts"
```

## Configuration

Everything lives in `conos.toml`. The default config ships with
three agents: concierge, sysadmin, and strategist.

```toml
[system]
name = "my-conspiracy"

[dashboard]
enabled = true
port = 8080

[contracts]
brief_output = "/srv/conos/context/system-state.md"

[base]
runner = "picoclaw"
provider = "anthropic"
api_key_env = "CONOS_API_KEY"     # env var holding your LLM provider key

[base.officer]
model = "claude-sonnet-4-6"

[base.operator]
model = "claude-sonnet-4-6"
```

The `[base]` section sets defaults. `[base.officer]` and `[base.operator]`
override per tier. Individual agents inherit from their tier unless they
specify their own `model` or `provider`.

### Adding an Agent

Append to `conos.toml`:

```toml
[[agents]]
name = "researcher"
tier = "worker"
mode = "on-demand"
instructions = """
You are the Researcher. You gather information from
approved sources and summarize findings.
"""
```

Redeploy to provision the new agent:

```bash
conos config apply    # if using conos
make deploy           # if building from source
```

Bootstrap runs on container start. It creates the Linux user, home
directory, inbox, systemd units, and ACLs. The agent exists because
Linux says it does.

### Agent Fields

| Field          | Required | Description                                    |
|----------------|----------|------------------------------------------------|
| `name`         | yes      | Agent name. Becomes Linux user `a-<name>`.     |
| `tier`         | yes      | `officer`, `operator`, or `worker`.            |
| `mode`         | yes      | `on-demand` (inbox-triggered) or `cron`.       |
| `instructions` | yes      | System prompt. Injected at task time.          |
| `roles`        | no       | List of roles. Loads skills from role dirs.    |
| `cron`         | no       | Cron expression. Required if `mode = "cron"`.  |
| `cli`          | no       | Alternative runtime binary.                    |
| `cli_args`     | no       | Arguments for alternative runtime.             |
| `packages`     | no       | Apt packages to install for this agent.        |

### Tiers

Tiers control privilege, not intelligence.

- **Officer** — sets policy. Broad read access, no direct system writes.
- **Operator** — executes. Can task other agents, modify config within scope.
- **Worker** — single-purpose. Reads own workspace, writes own outbox. Nothing else.

ACLs and sudoers enforce the boundaries. A compromised worker accomplishes
nothing beyond its intended capability.

## CLI Reference

### conos (host-side)

```
conos install                        # pull image, create env, start
conos start                          # boot the instance
conos stop [--force]                 # stop the instance
conos status                         # agent status overview
conos config apply                   # push config into running instance

conos agent list                     # list agents
conos agent <message>                # task the concierge
conos agent task <name> <message>    # task a specific agent
conos agent logs [-f] [-n N] [name]  # tail audit log
conos agent kill <name>              # stop a running agent
```

### conctl (inside the container)

You rarely need these directly. They are what `conos` calls over SSH.

```
conctl bootstrap                     # provision the conspiracy from config
conctl run <agent>                   # execute one task cycle for an agent
conctl status                        # agent status
conctl logs [-f] [-n N] [agent]      # audit log
conctl task [--agent <name>] <msg>   # drop a task into an inbox
conctl kill <agent>                  # stop agent systemd units

conctl healthcheck                   # evaluate all contracts
conctl doctor                        # system verification report
conctl brief                         # system state summary for agents
conctl manifest show                 # dump expected system state as YAML
conctl clear-sessions [agent]        # wipe PicoClaw session history

conctl protocol check <id> [--exempt <check> --reason <reason>]
conctl protocol list                 # list protocol contracts

conctl package install <pkg> --agent <name> [--save]
conctl package remove <pkg> --agent <name> [--save]
conctl package list [--agent <name>]

conctl artifact create|show|link|verify
conctl task-contract open|claim|update|show
```

### make targets (from the container repo)

```
make image          # build the container image
make deploy         # build + restart
make run            # start the container
make stop           # stop the container
make status         # agent health and last activity
make logs           # tail audit log
make responses      # latest response from each agent
make task MSG="..." # send a task to the concierge
```

## Inspecting a Running System

SSH into the container or use `docker exec`:

```bash
docker exec -it conos bash
```

Useful things to look at:

```bash
# Config
cat /etc/conos/conos.toml

# Agent home directories
ls /srv/conos/agents/

# Inboxes (where tasks land)
ls /srv/conos/agents/*/inbox/

# Outboxes (where responses appear)
ls /srv/conos/agents/*/outbox/

# Audit log
journalctl -u "conos-*" --no-pager -n 50

# Contract results
ls /srv/conos/contracts/results/

# System state brief (what agents see)
cat /srv/conos/context/system-state.md

# Who owns what
stat /srv/conos/agents/concierge/
```

Every agent is a Linux user. Every inbox is a directory. Every
permission boundary is a file mode. There is no abstraction layer
to debug — just the OS.
