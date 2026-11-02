---
title: "TCP Retransmission Explained with tcpdump"
description: "Use tcpdump to read TCP sequence numbers, follow ACKs, and investigate retransmissions without jumping to conclusions."
date: 2026-11-01 00:00:00 -0700
categories: [Network]
tags: [tcp, tcpdump, linux, packet-loss, troubleshooting]
---

A slow API call or a file transfer that keeps stopping may involve TCP retransmissions. Packet captures help us see what happened and where to investigate next.

This article uses simple examples to explain what tcpdump shows and how to read the evidence.

## 1. What does tcpdump do?

**tcpdump captures packets at a network interface.** It helps us check whether a host sends traffic, receives a reply, or sends the same data again.

For a TCP connection, we can look at:

- SYN, SYN-ACK, and ACK packets during the handshake.
- Sequence numbers that identify the bytes being sent.
- ACK numbers that show which bytes the receiver has acknowledged.
- Timestamps that show the gaps between packets.

tcpdump gives us evidence from the capture point. If a client sends SYN packets and gets no reply, that alone does not prove a firewall blocked them. We still need to check the forward path, the server, and the return path.

HTTPS encrypts application data, but TCP headers remain visible. We can inspect sequence numbers and ACKs without reading the HTTPS body.

## 2. What is TCP retransmission?

TCP uses sequence numbers to track bytes. The receiver sends acknowledgments, or ACKs, to confirm receipt.

When the sender believes some data is missing, it sends that data again. This is called **retransmission**.

Possible reasons include lost data, lost ACKs, or packets delayed long enough to trigger recovery. A retransmission alone does not tell us where the problem is.

Occasional retransmissions do not always cause a visible issue. Repeated recovery can increase response time and reduce throughput. Compare the capture with the time when users experienced the problem.

## 3. Capture the traffic

On the client, capture traffic to the affected server:

```bash
sudo tcpdump -i eth0 -nn -s 0 -w /tmp/tcp-retransmission.pcap \
  'host 203.0.113.10 and tcp port 443'
```

Replace `eth0`, the example IP, and the port with your actual values. `203.0.113.10` is a documentation address, not a test server. This filter includes both directions.

| Option | Purpose |
| :--- | :--- |
| `-i eth0` | Capture on this interface |
| `-nn` | Keep addresses and ports numeric |
| `-s 0` | Capture full packets up to tcpdump's snapshot limit |
| `-w` | Save packets to a file for later review |

Start before reproducing the issue, then press `Ctrl+C` to stop. Keep the capture short and focused. A capture file can contain application data, so store and share it carefully.

Read the file with:

```bash
tcpdump -nn -tttt -r /tmp/tcp-retransmission.pcap
```

`-tttt` shows the date and time for each packet. By default, tcpdump normally shows relative TCP sequence numbers after it has seen the connection's initial packets. Add `-S` when you need absolute sequence numbers, for example when comparing two captures.

The [tcpdump manual](https://www.tcpdump.org/manpages/tcpdump.1.html) explains these options.

## 4. Read sequence numbers and ACKs

The following lines are simplified examples, not a real incident capture. They show part of an established connection, using small relative sequence numbers.

```text
12:00:00.000000 client.50000 > server.443: Flags [P.], seq 1001:1501, ack 9001, length 500
12:00:00.020000 server.443 > client.50000: Flags [.], ack 1501, length 0
```

`seq 1001:1501` represents 500 bytes, from 1001 through 1500. The ending number is excluded.

`ack 1501` means the receiver has received all bytes before 1501 and expects that byte next. TCP ACKs are cumulative: one ACK can confirm several segments.

`Flags [P.]` means PSH and ACK are set. `Flags [.]` means ACK is set. These flags do not identify retransmissions.

The ACK confirms receipt by TCP. It does not confirm that the application finished processing the request.

## 5. Look for repeated data

Now consider this sequence:

```text
12:00:00.000000 client.50000 > server.443: Flags [P.], seq 1001:1501, ack 9001, length 500
12:00:01.000000 client.50000 > server.443: Flags [P.], seq 1001:1501, ack 9001, length 500
12:00:01.020000 server.443 > client.50000: Flags [.], ack 1501, length 0
```

The same byte range appears twice in the same direction and connection. This suggests retransmission. The original data may have been lost, or its ACK may not have reached the sender in time.

Look at the byte range, not only the packet length. Retransmitted bytes can be split into different-sized segments.

The one-second gap is only an example. TCP calculates its retransmission timeout from measured round-trip time and its variation, then backs off after repeated timer expiry. Do not assume every retransmission waits exactly one second. See [RFC 6298](https://www.rfc-editor.org/rfc/rfc6298.html).

## 6. Understand duplicate ACKs

If a receiver gets later data while an earlier part is missing, it may keep acknowledging the same next expected byte.

For example:

```text
Client sends bytes 1001:1501 -> Server receives them and sends ACK 1501
Client sends bytes 1501:2001 -> This segment is lost
Client sends bytes 2001:2501 -> Server sends duplicate ACK 1501
Client sends bytes 2501:3001 -> Server sends duplicate ACK 1501
Client sends bytes 3001:3501 -> Server sends duplicate ACK 1501
```

The repeated ACKs mean: "I am still waiting for byte 1501, even though later data has arrived."

Classic fast retransmit uses three duplicate ACKs to trigger recovery before the retransmission timer expires. See [RFC 5681](https://www.rfc-editor.org/rfc/rfc5681.html#section-3.2).

Modern TCP implementations can also use other loss-detection methods. Packet reordering can produce duplicate ACKs too, so repeated ACKs alone do not prove packet loss.

## 7. Compare both ends

A client capture shows what the client observed. Capture on the server too when possible, and compare the same connection and byte ranges.

On the server, filter for the client IP as the server sees it:

```bash
sudo tcpdump -i eth0 -nn -s 0 -w /tmp/server-tcp.pcap \
  'host 192.0.2.20 and tcp port 443'
```

Replace the example address and interface. If NAT is in the path, the server may see a different client address or port.

| Capture evidence | Next investigation |
| :--- | :--- |
| Data appears at the client but not at the server | Forward path, filtering, or capture gaps |
| Server sends an ACK that never appears at the client | Return path or capture gaps |
| Data and ACKs arrive, but the response is slow | Server processing and application dependencies, while checking the response path too |

Align the capture time windows. If the hosts' clocks differ, compare sequence numbers and packet details rather than subtracting their timestamps directly.

If a proxy terminates TCP, its backend connection is a separate connection with different sequence numbers. Investigate each connection separately.

## 8. Avoid common mistakes

### Repeated sequence numbers do not always mean retransmission

Pure ACKs can repeat sequence numbers without retransmitting data. Duplicate captures can also make one packet appear twice. Check the direction, data length, byte range, and timing together.

### A missing packet in a capture may be a capture problem

Check tcpdump's capture-drop count when stopping. Capture drops can hide part of the story. A zero count does not guarantee every packet was captured: a mirror port or another part of the capture setup may lose packets too.

### Wireshark labels are clues

Open the saved file in Wireshark and use this display filter:

```text
tcp.analysis.retransmission || tcp.analysis.fast_retransmission
```

These labels are analysis results, not TCP flags. Incomplete captures and packets arriving out of order can affect the interpretation. See [Wireshark TCP Analysis](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvTCPAnalysis.html).

tcpdump's normal text output does not automatically label every retransmission. Use it to collect packets and inspect sequence numbers; use Wireshark when you need help following a longer conversation.

## 9. Production checklist

1. Identify the affected connection and direction.
2. Capture the traffic before and during the problem.
3. Find repeated or overlapping data byte ranges.
4. Check whether ACKs advance and whether duplicate ACKs appear.
5. Compare captures from both ends and check for capture drops.
6. Check interface drops, firewall counters, congestion, and tunnel MTU based on the evidence.
7. Repeat the same test after the fix and compare retransmissions, response time, and errors.

A useful incident note records the affected connection, the repeated byte range, what each capture point saw, and how the result changed after the fix.
