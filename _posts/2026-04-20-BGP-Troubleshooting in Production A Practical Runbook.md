---
title: BGP Troubleshooting in Production: A Practical Runbook
date: 2026-04-20 09:00:00 -0700
categories: [Network]
tags: [bgp, routing, troubleshooting, runbook, production-network]
---

BGP issues can cause traffic loss, high latency, or route instability.
In production, we need a fast and clear method.
This document is a practical runbook for day-to-day BGP troubleshooting.

## 1) Start with impact

Before touching configs, confirm business impact:

- Which services are affected?
- Is impact global or only one region/site?
- Is traffic fully down or partially degraded?
- When did it start?

This helps you avoid blind changes.

## 2) Check BGP session state first

Core question: Is the BGP session up?

Common states:

- `Idle` / `Active`: session not established
- `Established`: session is up, move to route checks

Typical commands:

```bash
show bgp summary
show ip bgp summary
```

If the session is down, check:

- Peer IP reachability (`ping` / `traceroute`)
- TCP/179 blocked by ACL or firewall
- ASN mismatch
- BGP password mismatch
- `update-source` or loopback reachability
- TTL/security settings for eBGP multihop

## 3) If session is up, check route exchange

Session up does not mean routing is correct.

Check:

- Are prefixes received from peer?
- Are prefixes advertised to peer?
- Is the expected route in RIB/FIB?
- Is next-hop reachable?

Typical commands:

```bash
show bgp neighbors <peer> received-routes
show bgp neighbors <peer> advertised-routes
show ip route <prefix>
```

## 4) Validate policy and filtering

Many BGP incidents come from policy errors.

Focus on:

- Prefix-list or route-map deny by mistake
- Wrong community match
- AS-path filter too strict
- Max-prefix exceeded
- Local-pref or MED changed unexpectedly

Good practice:

- Check the recent config change first
- Diff current policy versus last known good config

## 5) Watch for route flap and instability

If routes keep changing, users see intermittent issues.

Check:

- BGP flap counters
- Interface errors/drops
- Upstream instability
- Aggressive timers
- CPU pressure on the control plane

Stabilization options:

- Dampening, used carefully
- Timer tuning
- Fix physical or link quality
- Coordinate with provider or NOC

## 6) Common root causes in real environments

- Wrong prefix-list update
- ASN typo during maintenance
- Firewall change blocking TCP/179
- Next-hop not reachable after IGP change
- Cloud route limit or max-prefix hit
- Missing rollback plan in the change window

## 7) Fast recovery checklist

When an incident is active:

1. Freeze non-essential changes
2. Restore last known good policy
3. Validate session state
4. Validate critical prefixes
5. Confirm data-plane recovery with real traffic
6. Keep monitoring for 30 to 60 minutes

## 8) Prevention

- Use a pre-change validation checklist
- Add automated config linting
- Monitor BGP sessions, prefix count, and route changes
- Keep clear rollback steps in every change ticket
- Run game-day drills for BGP failure scenarios

## Final note

In production, BGP troubleshooting is not only command output reading.
It is impact-first thinking, structured checks, and safe recovery.

If you follow the same runbook every time, MTTR will go down and confidence will go up.
