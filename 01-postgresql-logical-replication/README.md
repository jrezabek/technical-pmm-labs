# PostgreSQL Logical Replication and Failure Recovery

## Quick scan

**Built:** Two PostgreSQL 16 instances with logical replication  
**Tested:** Initial synchronization, subscriber outage, continued publisher writes, and automatic catch-up  
**Tools:** PostgreSQL 16, Docker, Docker Compose, PowerShell, VS Code

## What I built

I built a small PostgreSQL replication environment in Docker to get hands-on with concepts I usually encounter at a much higher level.

The setup had two PostgreSQL instances:

- a publisher that handled the original writes
- a subscriber that received replicated changes

I got logical replication working, confirmed the initial data made it to the subscriber, took the subscriber offline, kept writing to the publisher, then brought the subscriber back to see what happened.

Everything ran locally with fictional data. No employer, client, production, or unauthorized third-party systems were involved.

## What I actually did

I created a simple `orders` table on the publisher and added fictional records.

The subscriber had its own compatible `orders` table because logical replication does not automatically create the destination schema.

I created a publication on the publisher that made changes to that table available for replication, then created a subscription on the other PostgreSQL instance to consume them.

Once that worked, the existing rows appeared on the subscriber.

Then I stopped only the subscriber.

While it was offline, I added two more fictional orders to the publisher. The publisher kept accepting writes.

When I restarted the subscriber, the missing rows showed up without me manually copying anything over.

That was the piece I wanted to see for myself.

## A lot of tiny mistakes

This lab was also a crash course in how unforgiving database work can be.

I hit:

- missing semicolons
- malformed connection strings
- wrong database names
- incorrect passwords
- misspelled usernames
- Docker commands entered while I was still inside `psql`

I also broke the Compose file by putting the bottom-level `volumes` block in the wrong place.

The more useful lesson was learning to stop seeing every `ERROR` or `FATAL` message as the same problem.

A syntax error is different from an authentication failure. An authentication failure is different from a missing database.

Read what actually failed before randomly changing things.

## A few things that finally clicked

### Containers vs. volumes

This was one I had to slow down and understand.

**Container = disposable workspace**  
**Volume = persistent hard drive**

PostgreSQL runs inside the container. The database files live in the volume if I want them to survive the container being removed and recreated.

### Publisher vs. publication

These sound similar enough that I kept mixing them up.

The **publisher** is the PostgreSQL server containing the source data.

The **publication** is the PostgreSQL object that defines which table changes are available for replication.

### Subscriber vs. subscription

Same problem on the other side.

The **subscriber** is the PostgreSQL server receiving the replicated data.

The **subscription** is the configuration telling that server where to connect and which publication to consume.

### WAL

WAL stands for Write-Ahead Log.

The mental model that worked for me was:

**WAL is PostgreSQL's running journal of changes.**

PostgreSQL keeps recording what changed. If the subscriber disappears for a while, those changes do not just vanish. When it comes back, it can catch up on what it missed.

## What I took away from this

Before the outage:

```text
Publisher = in sync
Subscriber = in sync
```

During the outage:

```text
Publisher = still accepting writes
Subscriber = offline
```

After recovery:

```text
Publisher = current
Subscriber = caught up
```

## Evidence

The screenshots show:

1. Both PostgreSQL containers running
2. Replication working with the initial fictional records
3. Additional writes succeeding while the subscriber was offline
4. The subscriber caught up after it was restarted

## Why I did this

I work with technical infrastructure concepts as a product marketer.

There is a big difference between understanding the value proposition of replication and actually building it, breaking part of it, troubleshooting the setup, and watching recovery happen.

I am not trying to become a DBA.

I am trying to understand the technology underneath the messaging well enough to have better technical conversations.

## Safety and scope

This lab ran entirely in a local, isolated environment using fictional data.

The public version is scrubbed of credentials, personal details, machine-specific information, local paths, and anything tied to an employer or client.
