---
layout: home
title: ConspiracyOS
---

# ConspiracyOS

Every agent you deploy will eventually process adversarial input. A prompt injection in an email, a poisoned API response, a crafted webpage. The question is not whether an agent gets compromised. The question is what happens next.

If the answer depends on application-level guardrails — config flags, permission frameworks, safety wrappers — then what happens next is that the compromised agent ignores them. These are suggestions enforced by the same process that is now hostile. A locked door where the lock is inside the room.

ConspiracyOS takes a different position: the sandbox is the operating system. Each agent runs as a Linux user. Its permissions are `chmod`, not a policy file. Its network access is `nftables`, not a proxy allowlist. Its allowed commands are `sudoers`, not an application-layer filter. A compromised agent hits the kernel, not a config flag. It physically cannot read another agent's workspace, write to an inbox it lacks ACL permission for, or reach a network endpoint outside its firewall rules. Not because it was told not to — because the OS will not let it.

A fleet of agents coordinated this way is called a conspiracy. Agents are organized into tiers — officers set policy, operators execute, workers handle ephemeral tasks — and communicate by writing plain text files to each other's inboxes. systemd path units watch for new files and trigger the receiving agent. The entire coordination layer is filesystem and process management. There is nothing else.

## Contracts

Prevention is the OS's job. `chmod`, `nftables`, `sudoers` — these enforce boundaries at the kernel level, where a compromised agent cannot bypass them. Detection is the contract's job. Contracts are YAML-defined checks that run on systemd timers, asserting that the preventive controls stayed in place. An ACL was tightened? A contract verifies it stayed tightened. A sudoers rule was narrowed? A contract checks it every five minutes.

The [`contracts`](https://github.com/ConspiracyOS/contracts) framework is a standalone tool. It works outside ConspiracyOS, on any Linux system where you want YAML-defined checks running on a schedule. No dependency on the rest of the project.

## What this looks like

There is no dashboard. The state of the system is the filesystem:

```
$ ls /srv/conos/agents/
concierge/  sysadmin/  researcher/

$ stat -c '%U %a' /srv/conos/agents/sysadmin/workspace/
a-sysadmin 700

$ ls /srv/conos/agents/concierge/inbox/
001-abc.task  002-def.task

$ tail -1 /srv/conos/logs/audit/2026-03-10.log
2026-03-10T14:32:01Z [trust:verified] concierge routed task 002-def to sysadmin
```

`ls` is your agent dashboard. `stat` is your permission audit. `tail -f` is your observability stack. The tools already exist. They have for fifty years.

ConspiracyOS runs in a container on any Linux machine. `conctl` is the inner runtime. `conos` is the outer CLI on your host, an SSH wrapper around `conctl`. No cloud account, no platform dependency, no telemetry.

The [how it works](how-it-works) page covers the architecture in detail. Code is on [GitHub](https://github.com/ConspiracyOS).
