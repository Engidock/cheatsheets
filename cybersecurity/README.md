# Cybersecurity Fundamentals Cheatsheet

> Replace this placeholder with your actual EngiDock cybersecurity cheatsheet content.

## Core concepts

- **CIA triad** — Confidentiality, Integrity, Availability
- **Least privilege** — grant only the access a role actually needs
- **Defense in depth** — layer controls so one failure isn't total compromise

## Quick checklist for hardening a new server

1. Disable password auth for SSH, use keys only
2. Keep OS and packages patched (`apt update && apt upgrade` / equivalent)
3. Close unused ports (`ss -tulpn` to check what's listening)
4. Enable a firewall (`ufw` / `iptables`) with default-deny inbound
5. Set up centralized logging and alerting

---
*Full course: [engidock.com](https://www.engidock.com)*
