---
layout: page
title: Documentation
permalink: /docs/
---

# Getting Started with ConspiracyOS

## Prerequisites

- Linux (Ubuntu 22.04+ or Debian 12+ recommended)
- systemd
- Python 3.10+
- An OpenRouter API key (for LLM-backed agents)

## Install and Bootstrap

```bash
# Download and install the con binary
curl -L https://github.com/ConspiracyOS/agent-runner/releases/latest/download/con-linux-arm64 \
  -o /usr/local/bin/con && chmod +x /usr/local/bin/con

# Set your API key
echo "CON_OPENROUTER_API_KEY=your-key-here" > /etc/con/env

# Bootstrap the system (creates users, dirs, systemd units)
con bootstrap
```

## Your First Agent

Add an agent to `/etc/con/con.toml`:

```toml
[[agents]]
name = "my-agent"
tier = "worker"
mode = "on-demand"
instructions = """
You are my-agent. Describe what this agent does here.
"""
```

Then run `con bootstrap` to provision it.

## Your First Task

Drop a task file into the agent's inbox:

```bash
echo "Summarize the files in /srv/con/artifacts/" \
  > /srv/con/agents/my-agent/inbox/001-first-task.task
```

The systemd path watcher detects the new file and runs the agent.
Watch the audit log:

```bash
tail -f /srv/con/logs/audit/$(date +%Y-%m-%d).log
```
