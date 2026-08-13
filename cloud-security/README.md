# Cloud Security Cheatsheet

Complete quick reference guide for cloud security implementation.

## 1. Zero Trust Cloud Architecture

**Zero Trust Mantra:** "Never Trust, Always Verify"

```
VERIFY → ASSUME BREACH → SECURE LAYERS → LEAST PRIVILEGE → AUTHENTICATE
CONTINUOUSLY → MONITOR & VALIDATE
```

### Six Core Principles

- **Verify Explicitly:** Use all data points (identity, device, location, behavior)
- **Assume Breach:** Design with the assumption a compromise has already occurred
- **Secure Every Layer:** Equal security at app, network, data, and infrastructure levels
- **Least Privilege:** Minimum necessary permissions for a specific time/task
- **Continuous Verification:** Re-authenticate throughout the session, not just at login
- **Monitor & Validate:** Continuous security posture assessment and threat detection

### Quick Implementation Checklist

- [ ] Enable MFA/passwordless for all users
- [ ] Implement JIT/JEA (just-in-time / just-enough access)
- [ ] Configure microsegmentation at the network layer
- [ ] Encrypt data at rest (customer-managed keys) and in transit (TLS 1.3+)
- [ ] Deploy EDR/XDR for continuous endpoint monitoring
- [ ] Enable comprehensive audit logging (24+ months retention)
- [ ] Implement conditional access policies (device health, location, behavior)
- [ ] Conduct quarterly penetration testing

## 2. Threat Modeling & Risk Assessment

### STRIDE Framework

**S**poofing | **T**ampering | **R**epudiation | **I**nformation Disclosure | **D**enial of Service | **E**levation of Privilege

| Threat | Description | Mitigation |
|---|---|---|
| Spoofing (Identity) | Impersonating a user or service | MFA, service isolation |
| Tampering (Data/Code) | Modifying data or configuration | Encryption, digital signatures |
| Repudiation (Accountability) | Denying a performed action | Comprehensive logging |
| Information Disclosure | Data exposure to unauthorized parties | DLP, encryption |
| Denial of Service | Service unavailability | Rate limiting, auto-scaling |
| Elevation of Privilege | Gaining higher access than authorized | RBAC/ABAC, JIT access |

### Risk Scoring Formula

```
Risk = Likelihood x Impact
```

| Level | Likelihood | Impact | Action |
|---|---|---|---|
| Critical | High | Catastrophic | Immediate mitigation required |
| High | Medium-High | Major | Mitigate within 30 days |
| Medium | Medium | Moderate | Plan remediation within 90 days |
| Low | Low | Minor | Accept or mitigate opportunistically |

## 3. Advanced IAM & Privileged Access

### RBAC vs ABAC at a Glance

- **RBAC:** "Engineer can read code repos"
- **ABAC:** "Engineer can write to repos during work hours from corporate IP on a compliant device"

### Should This Access Be Granted? (Decision Tree)

```
Q1: Can user perform normal job duties with current permissions?
    NO  → Review and grant necessary permissions
    YES → Continue to Q2

Q2: Are permissions limited to what's necessary?
    NO  → Remove excess permissions
    YES → Continue to Q3

Q3: Is access time-limited?
    NO  → Implement JIT access expiration
    YES → ✔ APPROVED
```

### Privileged Access Management Essentials

- **Credential Management:** Vault, automatic rotation, no hardcoded secrets
- **JIT Access:** Request → Approval → Temporary Grant → Auto-revocation
- **Session Recording:** All privileged actions logged and recorded
- **MFA Required:** Multi-factor authentication for privileged operations
- **Approval Workflows:** Segregation of duties (requester ≠ approver)
- **Anomaly Detection:** Alert on unusual privilege usage patterns

```text
# Never: hardcoded credentials, SSH keys in code, env vars with secrets #
Always: managed identities (Azure), workload identity federation, Secrets Manager
```

## 4. Cloud Native Security Tooling

### Tool Categories & Key Functions

| Category | Tools | Key Function |
|---|---|---|
| Container Scanning | Trivy, Snyk, Anchore | Detect vulnerabilities in images before deploy |
| Runtime Security | Falco, Sysdig, Aqua | Monitor/enforce container behavior during execution |
| CSPM | Wiz, Prisma, Azure Security Center | Assess cloud config against benchmarks (CIS, PCI) |
| CWPP | Trend Micro, CrowdStrike, Palo Alto | Protect VMs, containers, and Kubernetes from exploits |
| EDR/XDR | CrowdStrike, SentinelOne, Microsoft Defender | Endpoint detection and response for advanced threats |
| SIEM | Splunk, ELK, Azure Sentinel | Centralize logs, correlate security events |

### Shift-Left Security Pipeline

```
Scanning in development (dev laptop) → Build pipeline (CI) → Registry → Runtime
```

### Alert Tuning Best Practices

- Define alert severity: only CRITICAL requires immediate response
- Whitelist known-good behavior to reduce false positives
- Correlate alerts: combine related signals into a single incident
- Set SLAs: CRITICAL (15 min), HIGH (1 hr), MEDIUM (4 hr), LOW (24 hr)
- Monthly review: analyze false positive trends and adjust thresholds
- Team training: ensure analysts understand alert context and response actions

## 5. Security Automation & SOAR

### Automation Maturity Levels

```
Level 1 (Manual) → Level 2 (Basic Scripts) → Level 3 (SOAR Playbooks) →
Level 4 (Autonomous) → Level 5 (Predictive)
```

### SOAR Core Functions

- **Orchestration:** Multi-step playbooks, conditional logic, human approval gates
- **Automation:** Connect hundreds of tools, normalize alert formats, enable cross-platform automation
- **Integration:** SIEM, EDR, CSPM feeds, cloud APIs, custom integrations
- **Response:** Threat intelligence enrichment, case management, SLA tracking

**SOAR ROI Metrics:** MTTR reduction (hours → minutes), Analyst time savings (40-60%), Cost per incident (down 90%)

### Playbook Design Principles

- Start with high-confidence, low-risk automations (block known malware)
- Always require approval for irreversible actions (data deletion)
- Implement a circuit breaker: pause if the automation error rate spikes
- Use dry-run mode: test playbook steps without making actual changes
- Version control playbooks: maintain history and the ability to roll back
- Monitor automation actions: keep an audit trail for accountability

## 6. Cloud Incident Response & Forensics

### NIST IR 5-Phase Cycle

```
1. Preparation → 2. Detection → 3. Containment → 4. Eradication → 5. Recovery
```

### Critical First Actions (First 1 Hour)

1. ✔ Assign Incident ID immediately
2. ✔ Activate incident response team
3. ✔ Preserve evidence (logs, snapshots, memory)
4. ✔ Isolate compromised resource
5. ✔ Block attack vector (firewall, disable account)
6. ✘ Do NOT shut down system without forensics
7. ✘ Do NOT delete logs or backup evidence

### Cloud Forensics Collection Checklist

- [ ] Memory dump from compromised instance
- [ ] EBS snapshot for disk forensics
- [ ] CloudTrail API audit logs (export to S3)
- [ ] VPC Flow Logs (network connections)
- [ ] Application logs and database audit trails
- [ ] IAM policy and role definitions
- [ ] Security group and network ACL rules
- [ ] Container image and log files
- [ ] Calculate SHA256 hashes (chain of custody)

**Cloud Forensic Challenges:** Ephemeral resources deleted automatically, logs rotate/delete within 30-90 days, shared responsibility model limits vendor cooperation.

## 7. Compliance-Driven Architecture

### Framework Quick Reference

| Framework | Scope | Key Requirements |
|---|---|---|
| ISO 27001 | Information Security | Access control, encryption, incident response |
| SOC 2 | Service Controls | Security, availability, integrity, confidentiality |
| HIPAA | Healthcare Data (PHI) | Encryption, BAAs, audit logs, access controls |
| PCI-DSS | Payment Card Data | Encryption, tokenization, network segmentation |
| GDPR | Personal Data (EU) | Consent, right to deletion, data residency |
| NIST CSF | Risk Management | Identify, Protect, Detect, Respond, Recover |

### Compliance Automation Commands

Enable comprehensive logging (AWS):
```bash
aws cloudtrail create-trail --name compliance-trail --s3-bucket-name compliance-logs
aws logs create-log-group --log-group-name /aws/compliance
```

Encrypt storage (Azure):
```bash
az storage account update --resource-group rg --name storage --encryption-services blob
```

Enable MFA (AWS):
```bash
aws iam enable-mfa-device --user-name username \
  --serial-number arn:aws:iam::123456789:mfa/device \
  --authentication-code1 123456 --authentication-code2 789012
```

### Compliance Checklist (Quarterly)

- [ ] Access review: verify users retain necessary permissions only
- [ ] Vulnerability scan: identify security gaps and remediation status
- [ ] Configuration audit: CSPM check for misconfigurations
- [ ] Incident review: no unresolved incidents beyond SLA
- [ ] Log verification: confirm logging enabled, retention sufficient
- [ ] Backup testing: recover from backup to verify data integrity
- [ ] Policy review: update compliance policies for new threats
- [ ] Audit trail review: examine logs for suspicious activity

## Quick Decision Trees

### Should This Action Be Automated?

```
Q1: Is action reversible?
    NO  → Requires human approval (data deletion, service termination)
    YES → Continue to Q2

Q2: Is there high confidence in detection?
    NO  → Requires analyst review (possible false positive)
    YES → Continue to Q3

Q3: Will automation reduce MTTR significantly?
    NO  → Manual response acceptable
    YES → ✔ AUTOMATE with monitoring
```

### Incident Severity Classification

| Severity | Response Time | Examples |
|---|---|---|
| CRITICAL | Immediate (0-15 min) | Active data breach, all systems down, ransomware detected |
| HIGH | 15-60 min | Partial outage, unauthorized access attempt, potential APT |
| MEDIUM | 1-4 hours | Misconfiguration detected, failed phishing attempt, suspicious pattern |
| LOW | 24 hours | Informational alerts, policy violation, documentation gap |

## Essential Commands Reference

### AWS Security Commands

```bash
# Check S3 bucket public access
aws s3api get-bucket-acl --bucket bucket-name

# List IAM policies for a user
aws iam list-user-policies --user-name username

# Enable logging for RDS
aws rds modify-db-instance --db-instance-identifier mydb \
  --enable-cloudwatch-logs-exports error,general,slowquery

# Get CloudTrail events
aws cloudtrail lookup-events --lookup-attributes \
  AttributeKey=EventName,AttributeValue=AssumeRole
```

### Azure Security Commands

```bash
# Check storage encryption
az storage account show --resource-group rg --name storage --query encryption

# List role assignments
az role assignment list --resource-group rg

# Enable diagnostic logs
az monitor diagnostic-settings create --name diagsetting \
  --resource /subscriptions/.../rg/providers/... \
  --logs '[{"category":"All","enabled":true}]'
```

### Kubernetes Security Commands

```bash
# Check RBAC configuration
kubectl get clusterrolebindings
kubectl auth can-i --list

# View pod security policies
kubectl get psp

# Check network policies
kubectl get networkpolicies --all-namespaces
```

## Critical Metrics to Track

### Key Performance Indicators

| Metric | Target | Benchmark |
|---|---|---|
| MTTD (Mean Time to Detect) | < 15 minutes | Industry avg: 1-4 hours |
| MTTR (Mean Time to Respond) | < 5 minutes | Industry avg: 4-24 hours |
| Vulnerability Resolution | Critical: 48h, High: 14d, Medium: 30d | Varies by framework |
| Access Review Completion | 100% quarterly | Compliance requirement |
| Patch Coverage | > 95% | Critical systems: 100% |
| Incident Resolution Rate | > 90% within SLA | Measures operational effectiveness |

## Red Flags & Warning Signs

### Immediate Action Required

- Public S3 bucket or Azure Blob with sensitive data
- Overly permissive IAM role (`*:*` permissions)
- Unencrypted database or storage
- No audit logging enabled
- Root/admin account used for daily operations
- Failed MFA for privileged operations
- Data exfiltration detected
- Suspicious cron jobs or scheduled tasks

### Good Security Signals

- MFA enabled for all users
- Encryption enforced (at rest & in transit)
- Comprehensive audit logging
- Regular access reviews completed
- No shared credentials or hardcoded secrets
- Incident response playbooks tested quarterly
- Security tools integrated and automated
- Annual third-party security audit completed

---
*Source: adapted from the Cloud Security cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
