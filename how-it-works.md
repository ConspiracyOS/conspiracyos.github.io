---
layout: page
title: How It Works
permalink: /how-it-works/
---

# How It Works

A conspiracy is a fleet of agents running inside a single container. Each agent is a Linux user — a real `uid` with a home directory, file permissions, and process isolation. Agents communicate by writing plain text files to each other's inboxes. There is no message bus, no RPC layer, no internal API. One agent tasks another by dropping a `.task` file in their inbox directory. That is the protocol.

## Tiers

Not all agents are equal. A conspiracy organizes agents into three tiers, each with different authority:

| Tier | Role | Authority |
|------|------|-----------|
| Officer | strategist | Sets policy, approves elevated actions |
| Operator | concierge, sysadmin | Routes tasks, executes operations |
| Worker | (ephemeral) | Runs a specific job, spawned on demand |

Tier determines what an agent can do. Officers can task anyone. Operators can task workers but not officers. Workers cannot task anyone — they receive work, do it, and write a response. These boundaries are not conventions. They are ACL rules on inbox directories, enforced by the kernel.

## The Coordination Layer

The entire coordination mechanism is five Linux primitives, each handling one concern:

**systemd path units** watch each agent's inbox. When a new file appears, systemd starts the agent's service. No polling, no daemon — the init system does what it was built to do.

**POSIX ACLs** (access control lists) determine which users can write to which directories. An operator can write to a worker's inbox because the ACL grants it. A worker cannot write to an officer's inbox because the ACL does not. `getfacl /srv/conos/agents/concierge/inbox/` shows exactly who has access.

**nftables** controls outbound network access per user. A worker agent that should only call one API endpoint gets a firewall rule scoped to its `uid`. Everything else is dropped. If the agent is compromised, it still cannot phone home — the kernel drops the packets before they leave the machine.

**sudoers** defines which commands an agent can run as root. A sysadmin agent can reload systemd units and manage users. A worker agent has no sudoers entry at all. The allowlist is explicit and auditable.

**File permissions** (`chmod 700`) on workspaces mean each agent's files are invisible to every other agent. No application-level access control. The filesystem enforces it.

## Contracts

A system that assumes breach needs continuous verification. Contracts are YAML-defined checks that run on systemd timers, asserting that the system matches its intended state:

```yaml
id: CON-SEC-001
description: Skill files must be owned by root
type: detective
checks:
  - name: skills_root_owned
    script:
      inline: |
        found=$(find /etc/conos/roles -name '*.md' ! -user root)
        [ -z "$found" ]
    on_fail: escalate
    severity: critical
```

This contract runs every five minutes. If an agent somehow modifies a skill file it should not own, the check fails and the system escalates. Contracts do not prevent drift — they detect it. Prevention requires trust in the thing being protected. Detection only requires `cron`.

The [`contracts`](https://github.com/ConspiracyOS/contracts) CLI is standalone. It runs on any Linux system, no dependency on the rest of ConspiracyOS.

## Inspecting a Running System

There is no dashboard and no proprietary observability stack. The system is the filesystem:

```bash
# List agents
ls /srv/conos/agents/

# Check an agent's inbox
ls /srv/conos/agents/concierge/inbox/

# See who can write to an inbox
getfacl /srv/conos/agents/sysadmin/inbox/

# Read the audit log
tail -f /srv/conos/logs/audit/$(date +%Y-%m-%d).log

# Check contract results
ls /srv/conos/contracts/results/
```

Every question about system state has a one-line answer using tools that ship with every Linux distribution.
