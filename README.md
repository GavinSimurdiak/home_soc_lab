# home_soc_lab

A personal Security Operations Center, built from scratch on my own hardware to get real, hands-on experience with the tools and workflows security analysts actually use — not just read about in coursework.

## Why I Built This

I'm a computer science student focused on cybersecurity, and I already had a foundation from CompTIA Security+ and my coursework in operating systems and computer architecture. What I didn't have was hands-on time with the actual tools: administering a SIEM, generating and reading real Windows security telemetry, simulating an attack against a target I control, and building a detection that actually catches it. This lab is me closing that gap, one documented cycle at a time.

## What This Project Demonstrates

- **Linux server administration** — installing, configuring, and operating Splunk Enterprise on Ubuntu Server, including recovering it after a kernel panic
- **SIEM operation** — Splunk Enterprise and Universal Forwarder setup, log ingestion, and writing search queries (SPL) from scratch
- **Windows security telemetry** — reading and interpreting Security Event Log data (logon types, failure codes, event correlation)
- **Attack simulation** — using Kali Linux and FreeRDP to generate real, controlled malicious activity against my own endpoint
- **Detection engineering** — writing and validating a Splunk search that correctly flags suspicious activity without false-positiving on normal traffic
- **Real troubleshooting** — diagnosing and fixing problems with no clear answer online (see the RDP scenario below), rather than following a tutorial step by step
- **Technical documentation** — writing up each experiment the way an analyst would document an investigation

## Architecture

Three virtual machines, isolated on their own VirtualBox network, each playing a distinct role in the loop: **attack → endpoint telemetry → SIEM → detection → documentation.**

```
   Kali Linux                Windows 11                Ubuntu Server
   (Attacker)      ────►    (Monitored Endpoint)  ────►   (SIEM)
                              generates Security         Splunk Enterprise
                              Event Log telemetry         + Universal Forwarder
```

All three VMs run on a single Windows 11 host laptop under Oracle VirtualBox, connected on an isolated NAT network so nothing here ever touches a real target.

## Tools & Technologies

| Category | Tool |
|---|---|
| Virtualization | Oracle VirtualBox |
| Attacker | Kali Linux |
| SIEM | Splunk Enterprise 10.4.1 (Ubuntu Server) |
| Telemetry collection | Splunk Universal Forwarder |
| Monitored endpoint | Windows 11 |
| Attack tooling | FreeRDP (`xfreerdp`) |
| Query language | Splunk SPL |

## Engineering Decisions

Not everything here went as originally planned, and I'm documenting that on purpose rather than only showing the parts that worked cleanly.

I initially set out to install Sysmon on the Windows endpoint for deeper process-level telemetry. I got it installed and generating events locally, but ran into a persistent issue getting those events into Splunk — narrowed down to the Universal Forwarder's service account lacking read access to Sysmon's restricted event channel. Rather than let that block progress indefinitely, I made the call to deprioritize it: the standard Windows Security log was already flowing reliably and gave me everything needed to build a real, working detection. Sysmon (or an equivalent using native Windows process-auditing) is a reasonable thing to revisit later, but it isn't gating the rest of the project. This is only one small example of the many issues/problems that I encounter during this journey, but utilizing documentation and various AI tools available, finding my way back onto the right path is easier than ever.

## Repository Structure

| Folder | Contents |
|---|---|
| `architecture/` | Diagrams and network topology for the lab and individual scenarios |
| `attack-scenarios/` | Write-ups of each simulated attack: what was run, what showed up in the logs, what was learned |
| `detection-rules/` | Splunk searches built and validated against real attack data |
| `reports/` | Setup and configuration notes for the lab infrastructure itself |
| `hardening-guides/` | Security improvements made to the lab based on what the attack scenarios revealed |
| `screenshots/` | Supporting evidence for each write-up |
| `scripts/` | Automation supporting the lab (attack scripts, setup helpers) |

## Status

This is an active, ongoing learning project, not a finished product — I'm building it out incrementally alongside coursework. Current focus is adding more attack scenarios and detections on top of the working Kali → Windows → Splunk pipeline.

