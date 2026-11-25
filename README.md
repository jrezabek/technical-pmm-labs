# Technical PMM Labs

I’m a product marketer. I’m not trying to become a DBA, network engineer, or SOC analyst.

But I’ve spent a lot of my career marketing fairly technical products, and at some point architecture diagrams, SME interviews, product demos, and technical briefs only get you so far.

Sometimes you need to actually touch the thing.

So that’s what this repo is.

These are hands-on labs where I build something, break something, test an assumption, chase an error, and try to figure out what the system is actually telling me.

The goal isn’t expertise.

It’s enough technical depth to ask better questions, understand what technical buyers are dealing with, and write about the problem without hand-waving through the hard parts.

## 1. PostgreSQL Logical Replication and Failure Recovery

I built a small PostgreSQL 16 replication environment in Docker with a publisher and subscriber.

Then I tested:

- initial logical replication
- subscriber outage
- continued writes to the publisher while the subscriber was offline
- subscriber restart
- automatic catch-up after recovery

This started as a replication lab and turned into a troubleshooting lab pretty quickly.

I broke YAML indentation, mixed up publications and publishers, mixed up subscriptions and subscribers, entered Docker commands from inside `psql`, and learned that staring at the word `ERROR` is much less useful than figuring out which layer is actually complaining.

The mental model that finally helped was simple:

**The WAL is basically a running journal.**

The publisher keeps recording changes. If the subscriber disappears for a while, those changes do not magically stop happening. When the subscriber comes back, it can catch up on what it missed.

The part I’ll remember is not the syntax.

It’s that one side of a replicated system can look perfectly healthy while the other side is having a very different day.

[View Lab 1](./01-postgresql-logical-replication/)

## 2. Network Discovery with Nmap

For this lab, I built an isolated Docker network with:

- Nginx
- Redis
- Netshoot

Then I used Nmap to test:

- host discovery
- default port scanning
- targeted port scanning
- service and version detection
- changes in reachability after modifying the environment

I expected three hosts.

Nmap found four.

That was my introduction to the Docker network gateway.

Then Redis did not show up the way I expected during a default scan, but it did when I explicitly looked for its port.

Same environment. Different question. Different observation.

Networking made more sense once I started picturing an apartment building:

- the network is the building
- the IP is an apartment
- the port is a door
- the service is whatever is behind that door
- Nmap is walking around checking which doors answer

Not perfect, but it gave me something concrete to hang the terminology on.

The bigger lesson was about interpretation.

A scan can tell me what responded.

It cannot automatically tell me why.

[View Lab 2](./02-nmap-network-discovery/)

## 3. Network Traffic Analysis with Wireshark

I created HTTP traffic between two containers, captured it with `tcpdump`, and then analyzed the conversation in Wireshark.

I looked at:

- live packet capture
- saved packet data
- the TCP three-way handshake
- the HTTP GET request
- the HTTP 200 response
- connection teardown

This one looked like nonsense for a while.

Not because I had never heard of TCP, HTTP, SYN, ACK, source ports, or destination ports. I understood those things individually.

I just wasn’t seeing the story.

Then I followed one simple exchange from beginning to end:

**SYN → SYN-ACK → ACK → GET → 200 OK → FIN**

That changed the whole thing.

Wireshark stopped looking like a spreadsheet full of networking trivia and started looking like a conversation history between two systems.

One system asks to talk.

The other agrees.

One asks for something.

The other sends it.

Then they wrap things up.

Obviously the protocols are more complicated than that, but that simple model was enough to make the packet sequence readable instead of abstract.

[View Lab 3](./03-wireshark-network-traffic-analysis/)

## 4. Phishing Email Investigation and Triage

I came into this one with more context than the others.

I previously worked in email security, so phishing concepts, social engineering, sender authentication, suspicious infrastructure, and reputation data were not new territory for me.

For this lab, I worked through a public, defanged phishing sample and reviewed:

- message content and social-engineering cues
- sender identity
- email authentication context
- SPF results
- indicators of compromise
- passive reputation data
- VirusTotal findings

What was different was actually walking through the investigation myself, step by step.

I understood the pieces at a product level. I had not personally worked through every part of the triage process in this format.

That exposed some useful nuance.

For example, SPF passed.

The useful question was not:

**“Does that make the email legitimate?”**

I already knew it did not.

The useful part was getting more precise about exactly what SPF was validating and what it was not.

Same thing with VirusTotal returning no detections.

A clean result is information.

It is not absolution.

The sample itself was intentionally straightforward. Real phishing investigations can involve far more context: message routing, identity compromise, payload analysis, user activity, authentication logs, campaign patterns, mail-flow telemetry, and other evidence that this lab did not attempt to reproduce.

Even in a simplified example, there was no single magic field that got to make the decision.

[View Lab 4](./04-phishing-investigation/)

## A pattern I did not plan

The same idea keeps resurfacing across very different technologies:

**What does the evidence actually prove?**

The publisher working does not prove the subscriber is healthy.

A scan result does not explain root cause.

A packet capture shows what systems exchanged, not necessarily why they behaved that way.

A reputation score is not a verdict.

That feels pretty relevant to product marketing.

Technical messaging gets sloppy when we jump from an observation to a conclusion because the conclusion makes for better copy.

These labs are forcing me to slow that down.

## What’s next

The first four projects are intentionally foundational.

The next ones need to get harder.

Planned next:

- Slow PostgreSQL query investigation
- Database locking and blocking investigation
- Security monitoring and incident investigation
- An integrated database and security capstone

I’m deliberately not trying to turn this into a giant collection of beginner labs.

The goal is progression: more ambiguity, more troubleshooting, more evidence, and better technical conversations.

## Safety and scope

All labs use local, isolated, fictional, or publicly available training data.

No employer, client, production, or unauthorized third-party systems are used.
