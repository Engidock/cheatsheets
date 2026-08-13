# Cybersecurity Cheatsheet

> Comprehensive quick-reference guide for all major security domains — 100+ checklists and frameworks covering IAM, cryptography, network security, vulnerability management, incident response, SIEM/detection, threat hunting, and governance/compliance.

## 🚀 Quick Start: Security Priorities

### First Things First (What to Implement First)

| # | Security Control | Effort | Impact | Why It Matters |
|---|---|---|---|---|
| 1 | Multi-Factor Authentication (MFA) | Medium (2–3 weeks) | High | Prevents 99% of account takeovers |
| 2 | Strong Password Policy + Manager | Low (1 week) | High | Foundation for all security |
| 3 | Backup & Recovery Testing | Medium (2–3 weeks) | High | Ransomware protection |
| 4 | Patch Management Program | Medium (3–4 weeks) | High | Prevents 80% of exploits |
| 5 | Security Training & Awareness | Low (1–2 weeks) | Medium | Prevents phishing attacks |
| 6 | Network Segmentation | High (4–6 weeks) | High | Limits breach damage |
| 7 | SIEM / Log Aggregation | High (4–5 weeks) | High | Enables threat detection |
| 8 | Incident Response Plan | Low (2–3 weeks) | High | Reduces response time |

### Immediate Actions (Do This Week)

- [ ] Enable MFA on critical accounts (email, cloud, admin)
- [ ] Implement a password manager for the team
- [ ] Test backup restoration (verify data integrity)
- [ ] Document current security inventory (what systems do you have?)
- [ ] Identify critical data (what needs the most protection?)
- [ ] Create an incident response contact list
- [ ] Schedule security training for the team

## 📋 Domain Quick-Reference Cheatsheets

### Identity & Access Management (IAM) — Quick Reference

**Principles:** Least privilege, separation of duties, need-to-know

**Key Controls:** MFA, password policy, access reviews, PAM, SSO

**Common Mistakes:** No MFA, privilege creep, no access reviews, shared accounts

**Metrics:** MFA adoption (target >95%), access review frequency (quarterly), privileged account age (<90 days)

**Quick Check:** Do critical accounts have MFA? Are access reviews current? Are privileges documented?

### Cryptography & Data Protection — Quick Reference

**Encryption Checklist:** Data at rest (AES-256), data in transit (TLS 1.2+), keys in KMS, rotation policy (annual)

**Key Management:** Never hardcode keys, use a Key Management System, separate keys by purpose, audit access

**Backup Strategy:** 3-2-1 rule (3 copies, 2 different media, 1 offsite), immutable backups, monthly restore test

**Common Failures:** Keys in code, no key rotation, untested backups, no encryption

**Quick Check:** Are keys in KMS? Is backup tested? Are encryption standards current? Do you rotate keys annually?

### Network Security — Quick Reference

**Segmentation:** DMZ (public), internal (users), data (sensitive), management (admins)

**Firewall Rules:** Default-deny policy, whitelist needed traffic, log all attempts

**Network Monitoring:** IDS/IPS for threats, NetFlow for anomalies, DNS monitoring for C2

**Common Gaps:** Flat networks, overly permissive rules, no monitoring, no encryption

**Quick Check:** Do you segment by risk? Are firewall rules documented? Is the network monitored? Do you have an IDS/IPS?

### Vulnerability Management — Quick Reference

**Scanning:** Weekly network scans, monthly authenticated scans, continuous cloud scanning

**Prioritization:** By CVSS score, exploitability, asset criticality, whether exploited in the wild

**Patching Timeline:** Critical 7 days, High 30 days, Medium 60 days, Low 90 days

**Metrics:** Mean time to patch (target <30 days), critical vuln age (target <7 days), patch compliance (>95%)

**Quick Check:** Do you scan regularly? Are critical vulns patched within 7 days? Is patch compliance tracked?

### Incident Response — Quick Reference

**Timeline:** Detection (0–1hr), Analysis (1–6hrs), Containment (1–24hrs), Eradication (1–7 days), Recovery (varies)

**Playbooks Needed:** Malware, ransomware, data breach, DDoS, account compromise, insider threat

**Key Actions:** Isolate systems, preserve evidence, notify stakeholders, investigate root cause

**Metrics:** MTTD <30 days; MTTR Critical <5min, High <30min, Medium <2hrs, Low <24hrs

**Quick Check:** Do you have documented playbooks? Is the IR team trained? Can you respond to the top 5 incident types?

### SIEM & Detection — Quick Reference

**Log Sources:** Firewalls, routers, servers, applications, cloud, endpoints, databases

**Alert Tuning:** Establish a baseline (normal behavior), alert on deviations, target <50% false positive rate

**Top Alerts:** Multiple failed logins, privilege escalation, unusual access patterns, large data transfers, lateral movement

**Retention Policy:** Critical events (2+ years), general logs (90 days), audit logs (1 year)

**Quick Check:** Do you have a SIEM? Are alerts tuned? Is false positive rate tracked? Are logs retained appropriately?

### Threat Hunting & EDR — Quick Reference

**Hunting Hypotheses:** Based on threat intel, TTPs from MITRE ATT&CK, industry attack patterns

**EDR Capabilities:** Process monitoring, file analysis, network connections, behavior detection, response actions

**Common Findings:** Lateral movement, persistence mechanisms, data exfiltration, C2 communication

**Frequency:** Weekly hunts (emerging threats), monthly planned hunts (hypotheses), ad-hoc (post-incident)

**Quick Check:** Do you hunt proactively? Is EDR deployed on critical endpoints? Are findings documented?

### Governance & Compliance — Quick Reference

**Frameworks:** NIST (what to do), CIS (how to do it), ISO 27001 (comprehensive program)

**Risk Assessment:** Probability × Impact, quarterly assessment, prioritized remediation

**Compliance Obligations:** GDPR (global), CCPA (California), HIPAA (healthcare), PCI (payments), SOC 2

**Audit Schedule:** Internal quarterly, external annually, board reporting quarterly

**Quick Check:** Do you know your compliance obligations? Are risks assessed? Is the audit schedule maintained?

## 🧩 Critical Security Frameworks & Checklists

### Risk Assessment Framework

```text
Risk = Probability x Impact

Probability: Low (1%), Medium (5%), High (25%)
Impact:      Low ($10K), Medium ($100K), High ($1M+)

Risk: Calculate for each vulnerability/threat

Decision Framework:
  Risk > Remediation Cost  -> Fix it
  Risk < Remediation Cost  -> Accept it (document decision)
  Risk Unknown             -> Research and investigate
```

### Security Maturity Model (NIST Levels)

| Level | Name | Characteristics | Typical Organization |
|---|---|---|---|
| 1 | Partial | Ad-hoc, reactive, no processes | Getting organized |
| 2 | Risk-Informed | Basic processes, some planning | Most SMBs |
| 3 | Repeatable | Standardized, documented, consistent | Sweet spot for most orgs |
| 4 | Managed | Metrics-driven, automated, monitoring | Enterprises |
| 5 | Optimized | Predictive, AI/ML, continuous improvement | Advanced programs |

### Incident Response Phases (NIST Framework)

1. **Detection** — Identify potential security incident (alert, report, anomaly)
2. **Analysis** — Verify incident, determine scope, assess severity (Critical/High/Medium/Low)
3. **Containment** — Stop spread, isolate systems, prevent further damage (tactical/strategic)
4. **Eradication** — Remove threat, patch vulnerabilities, improve controls
5. **Recovery** — Restore systems, verify clean state, document timeline
6. **Lessons Learned** — Conduct review, update playbooks, improve detection

### Crisis Decision Framework

1. **Gather Data** — What exactly happened? How many systems? Who accessed? What data?
2. **Assess Severity** — Is this an Incident (technical) or a Crisis (existential threat)?
3. **Activate Response** — War room, notify key stakeholders, determine lead
4. **Make Decision** — What's the best action? What's the trade-off?
5. **Communicate** — Tell customers, regulators, board, employees (coordinated message)
6. **Execute & Monitor** — Implement decision, watch for issues, adjust

### Security Control Implementation Checklist

- [ ] Define what success looks like (metrics, KPIs)
- [ ] Assess current state (what do we have?)
- [ ] Plan implementation (timeline, resources, dependencies)
- [ ] Design architecture (how will it work?)
- [ ] Pilot on a small group (test, get feedback)
- [ ] Document procedures (how to use, troubleshoot, maintain)
- [ ] Train users (ensure adoption)
- [ ] Roll out gradually (minimize disruption)
- [ ] Monitor metrics (is it working?)
- [ ] Tune/optimize (improve over time)
- [ ] Review quarterly (still meeting goals?)

## 🗓️ Daily, Weekly, Monthly Operational Checklists

### Daily Security Operations

- [ ] Review SIEM dashboard (any high-priority alerts?)
- [ ] Check system health (all systems operational?)
- [ ] Verify backup completion (did last night's backup complete successfully?)
- [ ] Check security metrics (alert volume, response times, etc.)
- [ ] Review overnight incidents (anything that happened during off-hours?)
- [ ] Validate logging (are all critical systems logging?)

### Weekly Security Operations

- [ ] Vulnerability scan review (new vulns? patch status?)
- [ ] Access review (any unusual access patterns?)
- [ ] Patch compliance report (what's not patched? why?)
- [ ] EDR review (any detections? investigation complete?)
- [ ] Firewall log review (blocked traffic normal? anomalies?)
- [ ] Team meeting (incidents, updates, priorities)
- [ ] Backup integrity test (restore one system from backup)

### Monthly Security Operations

- [ ] Security metrics review (MTTD, MTTR, alert quality, patch compliance)
- [ ] Access review completion (review all user access for necessity)
- [ ] Threat hunting planning (next month's hypotheses)
- [ ] Incident review (lessons learned from month's incidents)
- [ ] Compliance status (regulations, policies, standards)
- [ ] Vendor security review (are vendors meeting obligations?)
- [ ] Security training (current? new topics?)
- [ ] Budget review (spending on track?)
- [ ] Plan next improvements (what's the next priority?)

### Quarterly Security Operations

- [ ] Risk assessment update (risks changed? new threats?)
- [ ] Maturity assessment (where are we on the scale?)
- [ ] Roadmap review (are we on track? adjust priorities?)
- [ ] Metrics analysis (trends? improvements?)
- [ ] Board report (executive summary of status, risks, investments)
- [ ] Audit prep (are we audit-ready?)
- [ ] Security policy review (update needed?)
- [ ] Tool effectiveness review (are tools delivering value?)
- [ ] Team development (certifications, training, skills gaps)
- [ ] Plan next quarter (priorities, budget, resources)

### Annual Security Operations

- [ ] Comprehensive security assessment (maturity level, gaps, priorities)
- [ ] Incident response tabletop exercise (test IR capability)
- [ ] Penetration test (external, internal, cloud)
- [ ] Full access audit (who has access to what? remove unnecessary)
- [ ] Backup recovery drill (can we actually recover?)
- [ ] Disaster recovery test (can we restore after catastrophic failure?)
- [ ] Compliance audit (internal or external, prepare for external)
- [ ] Security training assessment (effectiveness, update content)
- [ ] Technology refresh planning (what's aging? what's vulnerable?)
- [ ] Budget planning for next year (investments, certifications, tools)

## 🛠️ Common Security Problems & Quick Solutions

### Alert Fatigue (100s/1000s of alerts daily)

**Problem:** Team ignoring alerts, missing real threats

**Quick Fix:**
- Assess false positive rate (how many are legitimate?)
- Identify top alert generators (which rules fire constantly?)
- Disable/tune noisy rules (reduce noise, keep signal)
- Establish baseline first (what's normal?)
- Alert only on deviations from baseline

**Target:** <50% false positive rate (or <25% if possible)

### Slow Response Time (Takes 2+ weeks to patch critical vulns)

**Problem:** Patch process is slow, critical systems unprotected

**Quick Fix:**
- Document current patch process (what are the steps?)
- Identify bottlenecks (where does it slow?)
- Automate where possible (scripting, tools)
- Parallelize (test & deploy simultaneously where safe)
- Prioritize critical systems (patch them first)

**Target:** Critical 7 days, High 30 days

### Detection Blind Spots (Attacker present 6+ months undetected)

**Problem:** No detection capability, attacks go unnoticed

**Quick Fix:**
- Enable logging on critical systems (servers, databases, cloud)
- Aggregate logs to SIEM (centralize for analysis)
- Create baseline (what's normal behavior?)
- Alert on anomalies (deviations from baseline)
- Hunt proactively (don't just react to alerts)

**Target:** MTTD <30 days (mean time to detect)

### Untested Backups (Assume backups work, until you need them)

**Problem:** Backups fail silently, no recovery when needed

**Quick Fix:**
- Test restore monthly (pick random system, restore from backup)
- Document restore procedure (steps, timeline, requirements)
- Verify data integrity (backup is actually complete?)
- Test recovery timeline (how long to restore a system?)
- Keep offsite copies (protect against data center disaster)

**Target:** 100% restore success rate

### No Incident Response Plan (Wing it when a breach happens)

**Problem:** When an incident occurs, chaos, inconsistent response

**Quick Fix:**
- Document IR process (who does what, when, how)
- Create playbooks for the top 5 incident types
- Define escalation (when to involve leadership, law enforcement, etc.)
- Plan communications (what to tell customers, regulators, board)
- Test with a tabletop exercise (practice the plan)

**Target:** Response time <1 hour from detection

### No Access Reviews (Privilege creep, old access remains)

**Problem:** Employees accumulate access, can access data they shouldn't

**Quick Fix:**
- Conduct access audit (who has access to what?)
- Validate necessity (does the user still need this access?)
- Remove unnecessary access (principle of least privilege)
- Schedule quarterly reviews (keep access current)
- Document approvals (who approved this access?)

**Target:** Quarterly reviews, 100% access coverage

## 👤 Role-Specific Security Checklists

### Security Analyst Checklist (Daily)

- [ ] Review SIEM alerts (prioritize by severity)
- [ ] Investigate high-priority alerts (determine if real threat)
- [ ] Document findings (ticket, timeline, actions taken)
- [ ] Escalate if needed (critical incidents to leadership)
- [ ] Update IR playbooks (based on findings)
- [ ] Communicate with team (incident status, next steps)

### System Administrator Security Checklist (Weekly)

- [ ] Check system logs (errors, security events)
- [ ] Verify patch status (critical updates applied?)
- [ ] Review user access (anything unusual?)
- [ ] Check backup completion (success or failure?)
- [ ] Update security tools (AV, EDR, firewall definitions)
- [ ] Review security alerts on managed systems

### Network Administrator Security Checklist (Weekly)

- [ ] Review firewall logs (blocked traffic normal?)
- [ ] Check IDS/IPS alerts (threats detected?)
- [ ] Monitor network performance (baseline normal?)
- [ ] Review VPN access (who's connected?)
- [ ] Update security policies (changes needed?)
- [ ] Test failover (security appliances working?)

### Security Manager/Leader Checklist (Monthly)

- [ ] Review all incidents from the month (trends, patterns)
- [ ] Assess team performance (metrics, incidents handled)
- [ ] Review security metrics (MTTD, MTTR, alert quality)
- [ ] Plan next improvements (what's the priority?)
- [ ] Communicate with leadership (status, risks, investments)
- [ ] Plan team development (training, certifications, career)

### CISO/Chief Security Officer Checklist (Quarterly)

- [ ] Review overall security posture (maturity assessment)
- [ ] Assess risks (what's the current risk profile?)
- [ ] Plan investments (budget allocation, priorities)
- [ ] Report to board (executive summary, metrics, risks)
- [ ] Review compliance status (regulations met?)
- [ ] Plan strategic initiatives (roadmap for next year)

## 📊 Critical Security Metrics Dashboard

### Detection & Response Metrics

| Metric | Definition | Target | Frequency |
|---|---|---|---|
| MTTD (Mean Time To Detect) | Days from breach to detection | <30 days (industry avg 200+ days) | Monthly |
| MTTR (Mean Time To Respond) | Time from detection to containment | Critical <5min, High <30min, Medium <2hrs, Low <24hrs | Monthly |
| Alert False Positive Rate | % of alerts that aren't real threats | <50% (or <25% if possible) | Weekly |
| Detection Rate | % of incidents detected by your tools vs. external customer reports | >75% (goal: you find it before customers report it) | Monthly |

### Patch & Vulnerability Metrics

| Metric | Definition | Target | Frequency |
|---|---|---|---|
| Critical Patch Age | Days since critical vuln published vs. patched | <7 days (attackers often exploit within days) | Weekly |
| Patch Compliance | % of systems with critical patches applied | >95% (some systems exempt, documented) | Weekly |
| Unpatched Critical Systems | Number of critical systems with known critical vulns | 0 (or documented exception with mitigation) | Daily |

### Access & Privilege Metrics

| Metric | Definition | Target | Frequency |
|---|---|---|---|
| MFA Adoption | % of users with MFA enabled | >95% (100% for admins, critical accounts) | Monthly |
| Access Review Currency | Last time access was reviewed | Quarterly (every 3 months) | Monthly |
| Privileged Account Age | Age of privileged account credentials | <90 days (rotate quarterly) | Monthly |

### Compliance & Governance Metrics

| Metric | Definition | Target | Frequency |
|---|---|---|---|
| Policy Currency | When was security policy last updated | Annually (review even if no changes) | Quarterly |
| Training Compliance | % of employees who completed security training | >95% (annual minimum) | Quarterly |
| Phishing Click Rate | % of employees clicking malicious links in test training | <5% (measure effectiveness) | Quarterly |

## 🎯 Key Takeaways

- Security is not one thing, it's a comprehensive program across 10+ domains.
- Start with MFA, strong passwords, backup testing, and patch management — highest impact for effort.
- Risk = Probability × Impact. Prioritize by risk, not by what's trending or flashy.
- Detection time is critical: industry average MTTD is 200+ days. Target <30 days.
- Response time matters: MTTR Critical should be <5 minutes. Have playbooks before an incident.
- Access is privilege: review quarterly, enforce least privilege, remove unnecessary access.
- Backups save the company: test monthly, keep an offsite copy, use immutable backups.
- Encryption is foundational: data at rest (AES-256), in transit (TLS 1.2+), keys in KMS.
- Logging is free intelligence: centralize logs, establish a baseline, alert on deviations.
- Patch 7/30/60/90: Critical within 7 days, High 30, Medium 60, Low 90. Automate where safe.
- Incident response plan matters: tabletop exercise annually, playbooks for top 5 threats, train the team.
- Maturity Level 3 is the sweet spot: standardized, documented, repeatable. Aim for Level 3+.
- Network segmentation: divide into DMZ, internal, data, and management networks. Whitelist traffic.
- Alert tuning is ongoing: <50% false positive rate is good, <25% is great. Tune based on baseline.
- Metrics drive improvement: MTTD, MTTR, patch compliance, MFA adoption, training rate. Track monthly.
- Teams prevent breaches: 80% of breaches involve human error. Training + culture = prevention.
- Compliance is table stakes: know your regulations (GDPR, CCPA, HIPAA, PCI), maintain an audit-ready state.
- Disaster recovery matters: test annually, document procedures, know RTO/RPO for critical systems.
- Threat intelligence informs decisions: follow industry trends, understand attacker TTPs (MITRE ATT&CK).
- Document everything: policies, procedures, decisions, exceptions. Audit trail matters for compliance.
- Prevention vs. detection: prefer prevention (stop it from happening), but detection is the backup when prevention fails.
- Regular assessments: quarterly reviews keep you honest, annual penetration tests show real gaps, tabletop exercises test response.
- Leadership alignment: CISO reports to board, investments prioritized, risks acknowledged, security is a business enabler.
- Culture matters: security is everyone's job. One vulnerable employee = one breach. Train, educate, communicate regularly.
- Continuous improvement: security is never done. Threats evolve, technologies change, priorities shift. Review and adjust regularly.

---
*Source: adapted from the Cybersecurity cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
