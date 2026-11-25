# Network Traffic Analysis with Wireshark

## Quick scan

**Built:** An isolated Docker network with a Netshoot client and Nginx web server
**Tested:** Live packet capture, TCP connection setup, HTTP request/response traffic, and saved PCAP analysis
**Tools:** Wireshark, tcpdump, Docker, Netshoot, Nginx, curl

## What I built

I built a small Docker network with two containers:

- a Netshoot client
- an Nginx web server

I generated a simple HTTP request between them, captured the traffic with `tcpdump`, saved the capture as a PCAP, and opened it in Wireshark.

Everything ran locally in an isolated lab environment.

No employer, client, production, or unauthorized third-party systems were involved.

## This one took me a minute

At first Wireshark looked like a wall of rows, ports, flags, numbers, and stuff I was apparently supposed to understand.

Trying to understand every packet independently was making it worse.

Once I stopped treating the packets like random events and started looking at the capture as one conversation, it clicked.

That was the shift.

## Watching the traffic live

I used `tcpdump` inside the Netshoot container and generated an HTTP request from another terminal using `curl`, a command-line tool for requesting a URL.

The capture showed:

- ARP traffic
- TCP connection setup
- an HTTP request
- an HTTP response
- connection teardown

The useful part was learning to stop staring at individual lines and instead read the sequence.

```text
CONNECT
   |
REQUEST
   |
RESPONSE
   |
DISCONNECT
```

## The conversation model

The instant-message analogy is what finally made this make sense for me.

```text
CLIENT                          SERVER

"Hey, are you there?"
SYN ---------------------------->

                         "Yep. You?"
                  <------ SYN-ACK

"Yep."
ACK ---------------------------->

"Send me the homepage."
HTTP GET ----------------------->

                         "Got it."
                  <---- 200 OK

"I'm done."
FIN ---------------------------->

                         "Got it."
                  <------ FIN/ACK
```

Ok, that would be an awful conversation, but thinking about it as a conversation helped me understand direction, sequence, connection setup, requests, responses, and connection state.

## Saving the traffic

Watching `tcpdump` live was useful, but I also wanted a saved capture I could inspect afterward.

I saved the traffic as a PCAP, generated another HTTP request with `curl`, then opened the capture in Wireshark.

The capture contained 13 packets matching the filter.

That did not mean I had 13 separate conversations.

It was one short conversation broken into individual packets.

Another thing that seems obvious after you understand it.

## Opening the PCAP in Wireshark

This was where I initially got lost.

Wireshark has a lot going on, and I was trying to understand the whole interface at once.

What I really needed was a much smaller question:

**What happened during this HTTP conversation?**

The sequence was:

```text
SYN
 ↓
SYN-ACK
 ↓
ACK
 ↓
GET /
 ↓
200 OK
 ↓
FIN / ACK
```

That made the packet list much easier to follow.

## IP addresses vs. ports

This also tripped me up briefly.

The IP address tells me **which system**.

The port tells me **which service on that system**.

So:

```text
client IP : ephemeral port
          |
          v
server IP : port 80
```

Port 80 was not "the destination."

The server IP was the destination system.

Port 80 was the HTTP service running on that system.

## tcpdump vs. PCAP vs. Wireshark

The way I think about it now:

**tcpdump = capture/watch the traffic**
**PCAP = saved recording of the traffic**
**Wireshark = inspect the recording**

Both `tcpdump` and Wireshark can analyze packets, but Wireshark made it much easier for me to visually follow the conversation after the traffic had already been captured.

## What I took away from this

The biggest lesson was:

**Packet analysis gives me evidence of what systems actually said to each other. It does not automatically explain why something happened.**

From this capture I could reasonably say:

- the client contacted the server
- a TCP connection was established
- the client sent an HTTP request
- the server returned `200 OK`
- the connection closed

I could not use this PCAP alone to say:

- the server had no vulnerabilities
- the entire network was healthy
- some unrelated problem was caused by the server
- I knew the root cause of every behavior I saw

That distinction between evidence and assumption keeps showing up in these labs.

I am starting to think that is the point.

## Cleanup

At the end of the lab, I removed both disposable containers and the isolated Docker network and verified that they were gone.

## Safety and scope

This lab was performed entirely against locally created systems in an isolated Docker environment.

The public version is scrubbed of credentials, personal details, machine-specific information, local paths, unnecessary infrastructure identifiers, and anything tied to an employer or client.

Packet capture should only be performed on systems and networks you own or are explicitly authorized to inspect.
