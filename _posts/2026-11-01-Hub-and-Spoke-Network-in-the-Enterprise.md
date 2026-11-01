---
title: "Hub-and-Spoke Networks in the Enterprise: Why This Simple Design Works"
date: 2026-11-01 09:00:00 -0800
categories: [Network]
tags: [hub-and-spoke, cloud-networking, enterprise-network, network-architecture]
---

Many companies today run their systems in more than one place. They use more than one public cloud. They have offices in different cities. Many of them also keep an old data center.

All of these places need to talk to each other. The question is: how do we connect them in a way that is simple, safe, and easy to grow?

One common answer is the hub-and-spoke design. This article explains what it is, why it works well, and what problems we need to watch for.

*Note: the company in this article is a simple example. It is not a real company's network.*

## 1. The problem: connect everything to everything

Imagine a company with many networks. If every network connects to every other network, we get a full mesh.

A mesh works when the number of networks is small. But it grows very fast. Each new network needs a new connection to all the old networks. Each place has its own routing tables and its own security rules.

Soon, no one can answer a simple question: "Which path does this traffic take?"

When something breaks, it is hard to find the problem. When we need a change, we have to change it in many places.

## 2. What is a hub-and-spoke network?

The design is simple:

- The **hub** is the center. It is one central network.
- Each **spoke** is one network that connects only to the hub. A spoke can be a cloud network, an office, or a data center.
- Spokes do not connect to each other directly. Traffic between two spokes goes through the hub.

```text
  Spoke A (cloud)  ──┐
  Spoke B (cloud)  ──┤                    Private
  Spoke C (office) ──┼── [ HUB ] ══════ backbone ══════ [ Other HUBs ]
  Spoke D (office) ──┤
     Data center   ──┘
     (firewall, DNS,
      shared services)
```

In this article, we talk about the private network inside the company, not the public internet. The services run on this private network, between the company's own sites.

Hubs in different regions connect to each other over a private backbone. A backbone is a fast, private connection, often provided by a network service provider.

Shared services, such as DNS and central firewalls, often live in the data center and serve all the spokes.

## 3. Why companies like this design

### 3.1 One place to set the rules

Security rules live in the hub. We write the policy one time, in one place. Every spoke follows the same policy.

This is much easier than writing the same rule in every network, and it keeps the rules the same everywhere.

### 3.2 Easy to add a new spoke

A new office or a new cloud network only needs one connection: to the hub. We do not need to touch the other spokes.

Adding a new place becomes a standard, repeatable job. The design can grow step by step, without big changes.

### 3.3 Spokes are separated by default

Two spokes cannot talk to each other directly. Their traffic must pass the hub, where we can check it, allow it, or stop it.

This gives us good separation between teams and between environments, for example between test systems and production systems. If one spoke has a problem, the problem does not easily spread to the others.

### 3.4 Shared services save money and time

Without a hub, each network often builds its own DNS, its own security tools, and its own connection to other sites. This costs money and takes time to manage.

With a hub, we build these shared services one time, in one central place, and all spokes use them over the private network.

### 3.5 Problems are easier to find

Traffic always takes a clear path: spoke, then hub, then spoke. The hub sees most of the traffic, so it is a good place for logs and monitoring.

When a user says "the network is slow," we know where to look first. A clear path means faster troubleshooting.

## 4. One hub is not always enough

A large company may have systems in many parts of the world. One single hub for the whole world is not a good idea. Traffic may need to travel a very long way just to reach the hub.

So large companies often use more than one hub:

- One hub for each region.
- Spokes connect to the hub in their own region.
- The regional hubs connect to each other over a private backbone.

In some designs, traffic between hubs also passes through a central firewall, often in a data center, for a security check before it goes to the other region. This keeps the same security rules for traffic inside one region and traffic between regions.

This is still the same idea. We just use it at two levels: spokes connect to a regional hub, and regional hubs connect to each other.

## 5. The costs and the risks

This design is not perfect. We need to be honest about its weak points.

- **The hub is very important.** If the hub goes down, many spokes lose their connection. So the hub needs backup systems and more than one path.
- **The hub can become busy.** All spoke-to-spoke traffic goes through it. We need to plan its bandwidth and watch its load.
- **The path can be longer.** Two spokes in the same city may send traffic to a hub far away and back. The same is true when traffic between regions must first go to a central firewall for a check. That check adds safety, but it also adds time and load. For traffic that needs very low latency, we may need a direct connection as a special case.
- **Cost and complexity move to the hub.** The design does not remove hard problems. It moves them to the center, where we must design routing and firewalls with care. For example, traffic must take the same path in both directions when it passes a firewall.

## 6. A short checklist

A hub-and-spoke design is often a good choice when:

- The company has many networks, and the number keeps growing.
- The company uses more than one cloud, plus offices and a data center.
- Security policy must be the same in all places.
- Teams need clear separation between networks.
- Fast troubleshooting and clear logs are important.

It may not be a good choice when two systems need the lowest possible latency between them, all the time. In that case, a direct connection can be added for that special traffic.

## 7. Summary

The hub-and-spoke design is popular because it is simple to understand and simple to grow:

- One center holds the shared services and the security rules.
- Each new network connects in one standard way.
- Traffic paths are clear, so problems are easier to find.

The key work is in the hub: build it with backup, plan its bandwidth, and design its routing with care. If we do that well, this old and simple design can support a large and modern company network.

---

*Views are my own. The example in this article is illustrative and does not describe any real company's network.*
