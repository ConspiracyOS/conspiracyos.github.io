---
layout: page
title: How It Works
permalink: /how-it-works/
---

# How ConspiracyOS Works

## The Conspiracy Model

A **conspiracy** is a fleet of agents coordinated using Linux OS primitives.
Each agent is a Linux user. Agents communicate by writing files to each
other's inboxes. All state is on disk. Everything is inspectable with
standard Linux tools.

### Agents, Inboxes, Tiers

Agents are organized into three tiers:

| Tier | Who | Authority |
|------|-----|-----------|
| Officer | ceo, cto, cso | Sets policy, approves elevated actions |
| Operator | concierge, sysadmin | Executes, routes, operates |
| Worker | (ephemeral) | Runs specific tasks, spawned on demand |

To send a task to another agent, write a plain text file to their inbox:
`/srv/con/agents/<name>/inbox/<NNN>-<id>.task`

### Linux-Native

ConspiracyOS uses the OS as its coordination layer:

- **Filesystem** — inboxes, workspaces, artifacts, audit logs
- **systemd** — path watchers trigger agents when inbox changes
- **ACLs** — enforce who can task whom
- **nftables** — enforce what each agent can reach on the network
- **sudoers** — enforce what each agent can execute

### Inspectability and Auditability

Every significant action is logged to `/srv/con/logs/audit/`.
Every capability is enforced by the OS, not by instructions.
A compromised agent cannot exceed its Linux permissions.

```bash
# See what agents exist
ls /srv/con/agents/

# See what's in an agent's inbox
ls /srv/con/agents/concierge/inbox/

# See the audit log
tail -f /srv/con/logs/audit/$(date +%Y-%m-%d).log

# See active contracts
ls /srv/con/contracts/
```
