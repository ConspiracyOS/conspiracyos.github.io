---
layout: page
title: Documentation
permalink: /docs/
---

# Getting Started with ConspiracyOS

## Prerequisites

- Linux (Ubuntu 22.04+ or Debian 12+ recommended)
- systemd
- Go 1.22+ (to build from source)
- An OpenRouter API key

## Install and Bootstrap

ConspiracyOS runs as a container with systemd as PID 1. You need Docker (or Podman)
and Go 1.22+ to build from source.

```bash
# Clone the inner runtime and container build repos
git clone https://github.com/ConspiracyOS/conctl.git
git clone https://github.com/ConspiracyOS/container.git

# Build the conctl binary (runs inside the container)
cd conctl && go build -o conctl ./cmd/conctl/ && cd ..

# Set your API key
cd container
cp .env.example srv/dev/container.env
# Edit srv/dev/container.env: set CONOS_API_KEY=your-key-here

# Build the container image and deploy
make image CONCTL_BIN=../conctl/conctl
make deploy
```

The container boots with systemd as PID 1, provisions all agent users and inboxes,
and starts listening for tasks.

Verify the system is up:

```bash
make status
```

## Your First Task

Send a task to the concierge:

```bash
make task MSG="What agents are running in this conspiracy?"
```

The concierge picks it up from the outer inbox, determines the right agent, and routes it.
Watch the audit log to see activity:

```bash
make logs
```

Check for responses in agent outboxes:

```bash
make responses
```

## Useful Commands

```bash
make status         # show each agent's health and last activity
make logs           # tail the audit log (last 20 lines)
make responses      # show latest response from each agent's outbox
make task MSG="..." # send a task to the concierge
make deploy         # rebuild the image and restart the container
```

## Adding a Custom Agent

Add an entry to `configs/default/conos.toml` in the `container` repo:

```toml
[[agents]]
name = "my-agent"
tier = "worker"
mode = "on-demand"
instructions = """
You are my-agent. Describe what this agent does here.
"""
```

Then rebuild and redeploy:

```bash
make deploy
```

Bootstrap runs automatically on container start and provisions any new agents
defined in the config.
