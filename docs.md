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

```bash
# Clone the repository
git clone https://github.com/ConspiracyOS/agent-runner.git
cd agent-runner

# Build the binary (requires Go 1.22+)
make linux           # amd64; use `make linux-arm64` for arm64
sudo cp con /usr/local/bin/con

# Set your API key (mode 600, readable only by root/systemd)
sudo mkdir -p /etc/conos
printf 'CONOS_API_KEY=your-key-here\n' | sudo tee /etc/conos/env >/dev/null
sudo chmod 600 /etc/conos/env

# Copy the default config
sudo cp configs/default/conos.toml /etc/conos/conos.toml

# Bootstrap the system (creates users, dirs, systemd units — run once as root)
sudo con bootstrap
```

Verify the system is up:

```bash
con status
```

## Your First Task

Send a task to the concierge:

```bash
con task "What agents are running in this conspiracy?"
```

The concierge picks it up from the outer inbox, determines the right agent, and routes it.
Watch the audit log to see activity:

```bash
con logs
```

Check for responses in agent outboxes:

```bash
con responses
```

## Useful Commands

```bash
con status          # show each agent's health and last activity
con logs            # tail the audit log (last 20 lines)
con responses       # show latest response from each agent's outbox
con healthcheck     # evaluate all system contracts
```

## Adding a Custom Agent

Add an entry to `/etc/conos/conos.toml`:

```toml
[[agents]]
name = "my-agent"
tier = "worker"
mode = "on-demand"
instructions = """
You are my-agent. Describe what this agent does here.
"""
```

Then run `sudo con bootstrap` to provision it. Bootstrap creates the Linux user,
home directory, inbox, workspace, and systemd path unit for the new agent.
