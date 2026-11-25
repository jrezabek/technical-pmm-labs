# Phishing Email Investigation and Triage

## Quick scan

**Built:** A safe phishing-triage workflow using a public, defanged email sample  
**Tested:** Message content, sender identity, email authentication context, IOC extraction, and passive reputation checks  
**Tools:** VS Code/browser text view, VirusTotal, public defanged email corpus

## What I did

I analyzed a public, defanged phishing email sample and worked through a basic triage process.

I have worked around phishing security before, so things like urgency, suspicious sender identities, and sketchy calls to action were familiar. What I wanted from this lab was a closer look at the mechanics: what the headers actually tell me, what I can learn from the link and sender information, and how much weight I should give reputation tools.

The email looked suspicious. VirusTotal basically shrugged.

That turned out to be one of the more useful parts of the exercise.

## First pass: does this thing look weird?

Before touching VirusTotal or digging through the headers, I looked at the email itself.

The message leaned heavily on fear and urgency. It referenced compromised information, pushed the recipient to take swift action, and included a link to supposedly secure the device.

The sender also looked off. The display name made the message look like a payment or security alert, but the sender domain did not appear to have any obvious connection to the company or service the email was trying to represent.

None of that proves phishing by itself, but there were enough things that did not line up to justify looking closer.

## Header analysis

This was probably the most technical part of the lab.

The email passed SPF, which stands for **Sender Policy Framework**. In plain English, SPF checks whether the mail server sending a message is authorized to send email for the domain being checked.

That sounds reassuring, but an SPF pass does not mean:

**“This email is legit.”**

In this sample, the domain SPF checked was different from the visible `From:` domain the recipient would actually see. So the message could pass SPF while still presenting a suspicious sender identity.

That helped tighten up my understanding of email authentication. The result is useful, but only if I understand exactly what it is validating.

## Looking at the link

The message wanted the recipient to click a link, so I looked at where the link was actually pointing instead of trusting the text around it.

The destination used legitimate shared cloud infrastructure. That does not automatically make the link safe, just like using a recognizable cloud provider does not make the provider itself suspicious. Legitimate services can be abused too.

The useful question was not just, “Do I recognize this domain?” It was whether the actual destination made sense in the context of the sender, the message, and what I was being asked to do.

## IOC extraction

I pulled out the indicators that were actually useful to investigate:

- visible sender domain
- SMTP mail-from domain
- sending IP
- defanged destination URL

I did not try to turn it into a giant indicator list just to make the investigation look more complicated than it was. The point was to identify the pieces of the message that could actually help me decide whether something was wrong.

## Reputation checks

I checked the sender-side indicators in VirusTotal and got no current vendor detections. The community score was neutral too.

That was interesting because the message itself still looked suspicious.

A clean reputation result does not mean an indicator is safe. It means the sources participating in that lookup were not currently flagging it. New, low-volume, or short-lived phishing infrastructure may not have much reputation history yet.

So VirusTotal gave me another piece of evidence, but it did not make the decision for me.

## Assessment

Taken together, I would classify the message as **likely phishing with high confidence**.

The conclusion came from the combination of the suspicious sender identity, the mismatch in the authentication details, the fear and urgency in the message, the push to click a link, and where that link was actually pointing.

No single indicator made the call. The combination did.

## What I took away from this

This was a deliberately obvious training example. Real phishing emails are getting much harder to spot.

They can be polished, use legitimate services, pass some authentication checks, and point to infrastructure you recognize. Reputation tools may not have anything useful to say yet either.

For me, the practical takeaway is to slow down when something feels off. Check who actually sent the message and where a link really goes before clicking it. Treat unexpected attachments carefully. If a message claims to come from someone you know but something does not line up, verify it with them through another channel.

And if an email is asking you to do something you were not expecting, especially involving credentials, money, security, or urgency, it is worth taking a minute to investigate before acting.

The technical tools help, but they are still only part of the decision.

## Safety and scope

This lab used a public, defanged email sample. I did not re-fang or click suspicious URLs, open attachments, enter credentials, or interact with attacker-controlled infrastructure.

The message was inspected as text only, reputation checks were passive, and no work, client, production, or personal email data was used.
