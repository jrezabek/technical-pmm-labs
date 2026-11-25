# Network Discovery with Nmap

## Quick scan

**Built:** An isolated Docker network with Nginx, Redis, and Netshoot  
**Tested:** Host discovery, targeted port scanning, service detection, and changes in reachability  
**Tools:** Nmap, Docker, Netshoot, Nginx, Redis

## What I built

I built a small Docker network so I could learn Nmap without scanning anything I did not own or have permission to touch.

The environment included:

- an Nginx web server
- a Redis server
- a Netshoot container running Nmap

Everything ran locally on an isolated Docker network.

The goal was not to memorize a giant list of Nmap flags. I wanted to understand what Nmap was actually telling me, what it was not telling me, and how the results changed when I changed the environment.

No external, work, client, production, or unauthorized third-party systems were scanned.

## Finding out what was actually on the network

I started with host discovery.

I expected three hosts because I had created three containers.

Nmap found four.

**Well, shit.**

There was definitely a brief moment of, “What did I screw up?”

So instead of assuming Nmap was wrong, I checked the scanner's network and routing information to figure out what else was there.

The fourth host was Docker's network gateway.

That was my first useful lesson from this lab:

**What I intentionally deployed is not necessarily everything that exists on the network.**

## Ports took me a minute

I ran a normal Nmap scan and initially misread the output.

Nmap showed hundreds of TCP ports as closed.

For a minute my brain translated that into hundreds of services.

That is not what it meant.

Nmap was reporting the results of the ports it checked.

The web target was easy to spot because HTTP was open.

Redis was more interesting.

I knew Redis was running, but its service did not appear as open in the default scan.

That forced an important concept to click:

**"No open ports found" does not mean "no open ports exist."**

It means Nmap did not find an open port among the ports that particular scan tested.

So I explicitly told Nmap to check the Redis port, then did the same thing for the web target.

That produced a different result because I was asking a different question.

## When it clicked

This was the analogy that finally made the commands click for me:

**Network = apartment building**  
**IP address = apartment**  
**Port = numbered door**  
**Service = what is answering behind the door**  
**Nmap = security guard checking the building**

So a targeted service scan roughly translates to:

> Go to this apartment, check this specific door, and if someone answers, try to figure out who or what is behind it.

Not technically perfect, but it worked.

The `-p` option tells Nmap which door I want checked.

The `-sV` option tells Nmap to investigate the service or software answering behind an open door.

## Establishing a baseline

Once I knew what the normal environment looked like, I had a basic baseline.

Then I intentionally changed the environment and repeated the same targeted test.

Before the change, the web service was open and identifiable.

After the change, the same port showed as filtered and Nmap could no longer identify the Nginx version.

That was useful, but the more important part was interpreting it carefully.

If I only had the scan results, I could say:

**Something about reachability changed.**

I could not say:

- Nginx definitely crashed
- a firewall definitely changed
- the host definitely went offline
- I knew the root cause

Those might be possible explanations.

The scan alone did not prove them.

## What I took away from this

- `-sn` helps answer "who is here?"
- a default scan does not mean every possible port was tested
- `-p` lets me explicitly choose which ports to investigate
- `-sV` attempts to identify what is answering behind an open port
- open, closed, and filtered mean different things
- negative results only apply to what I actually tested
- discovery results are evidence, not automatic root-cause analysis

The biggest lesson:

**A scan tells me what I observed. It does not automatically tell me why I observed it.**
