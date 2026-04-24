# Device Hardening

This page describes the CPE security hardening baseline that RansNet applies to Customer Premises Equipment (CPE) such as SD-WAN gateways and branch routers (CMG, HSG, HSA, UA, XE, UAP) to comply with industry best practices, including CIS Benchmarks where applicable, during device onboarding, provisioning, and firmware updates — ensuring devices are securely configured before deployment.

---

## What are CIS Benchmarks?

CIS Benchmarks are security hardening standards developed by the Center for Internet Security (CIS). They outline detailed, widely accepted configuration recommendations for systems ranging from operating systems (e.g. Linux, Windows) to network devices and applications.

For embedded CPE devices, not all CIS controls apply directly. RansNet CPE device OS/image are pre-built with functionally equivalent controls where native CIS rules cannot be applied due to platform constraints. These benchmarks serve as a baseline for secure configuration that auditors, integrators, and service providers often reference.

RansNet CPE hardening aligns with the following standards:

| Standard | Version | Scope |
|---|---|---|
| CIS Benchmark for Linux | Level 1 – Enterprise | Sections applicable to embedded systems |
| CIS Benchmark for Network Devices | v2.0 | Supplementary |
| CIS Critical Security Controls | v8 | Framework reference |
| ISO/IEC 27001 | 2022 | Annex A — information security controls |
| NIST SP 800-53 | Rev 5 | Control families: AC, AU, IA, SC, SI |

---

## Purpose

This hardening baseline aims to:

- Reduce the device attack surface
- Enforce secure defaults
- Protect against common network attacks (e.g. SYN floods, brute force, unauthorized access)
- Improve resilience against denial-of-service threats
- Ensure consistent security posture across all deployed devices

---

## Hardening Controls

The controls below highlight critical requirements as part of CPE deployments.

Some controls are **OS defaults** (built into the OS, already CIS-compliant, and cannot be changed). The remaining controls require action as part of the hardening process.

### 1. Operating System and Services

| Control | Action Required |
|---------|----------------|
| Default credentials must be changed before onboarding | Change default password at first login |
| Passwords must meet complexity requirements | Minimum 12 characters; must include uppercase, lowercase, digit, and special character; must not match username or hostname |
| Password reuse is restricted | OS enforces no reuse of the last 5 passwords |
| Account lockout is enforced | Account is locked after 5 consecutive failed login attempts; auto-unlock after 15 minutes |
| Unused services (e.g. telnet, FTP) are disabled | OS default |
| Privileged access (e.g. root/enable) is restricted | Ensure login and enable passwords are different and comply with corporate security policy |
| Login warning banner is displayed | Configure a legal warning banner on all management interfaces (SSH, console, web). See [example](#login-warning-banner) below. |
| Firmware is kept up-to-date | Upgrade to latest STABLE firmware after onboarding; apply security patches within the timelines defined in [Vulnerability and Patch Management](#vulnerability-and-patch-management) |

### 2. Management and Access Controls

| Control | Action Required |
|---------|----------------|
| Management access (SSH/HTTPS) enabled only over secure channels | Configure VRF for management interface; add firewall-input rules to permit authorized source IPs only |
| Telnet and unencrypted management interfaces are disabled | OS default |
| SSH root login is disabled | OS default |
| SSH protocol version 2 only | OS default |
| SSH authentication attempt limit | OS default. `MaxAuthTries 3` configured; brute-force attempts trigger session termination. |
| SSH idle session timeout | OS default. Sessions terminated after 10 minutes of inactivity (`ClientAliveInterval 300`, `ClientAliveCountMax 2`) |
| Idle CLI session timeout | OS default |
| SNMP v1 and v2c are disabled | OS default — SNMP v1/v2c are not enabled. If SNMP monitoring is required, use SNMPv3 with `authPriv` mode (SHA-256 authentication, AES-256 encryption) only |

### 3. Network and Firewall Hardening

| Control | Action Required |
|---------|----------------|
| Stateful firewall is enabled | OS default |
| Default deny rules apply to untrusted interfaces | OS default |
| Management and control ports restricted by ACLs | Configure VRF for management interface; add firewall-input rules to permit authorized source IPs only |
| Address translation and NAT used responsibly | Review firewall rules — strictly no "permit all" type rules |
| Rate limiting for TCP SYN, ICMP, and other flood vectors | Configure rate limiting for management (SSH/HTTPS) services if these are allowed |
| Unused physical ports are administratively disabled | Disable switch ports and WAN interfaces that are not in use |

### 4. Cryptography and TLS

| Control | Action Required |
|---------|----------------|
| Only TLS 1.2 and TLS 1.3 are enabled | OS default |
| Weak protocols (SSL, TLS 1.0, TLS 1.1) are disabled | OS default |
| Cipher suites adhere to strong cryptographic practices (forward secrecy, AEAD) | OS default |
| Certificate validation follows industry best practices | Review company SOP; use valid SSL certificates and rotate yearly |
| VPN and IPSec use strong algorithms | IKEv2 only; AES-256, SHA-256 minimum; DH group 14 (MODP-2048) or higher |

### 5. Logging and Monitoring

| Control | Action Required |
|---------|----------------|
| System and security logging is enabled | Enable logging for firewall rules; export logs to a syslog collector |
| Authentication attempts, firewall drops, and config changes are logged | Deploy syslog collector to receive logs from all devices; severity level `info` and above |
| Configuration changes are recorded with operator identity and timestamp | OS default. Ensure AAA accounting is enabled; all CLI commands executed under a named account are forwarded to syslog |
| NTP time synchronization configured for consistent timestamps | Use customer's internal NTP server if available; configure at least two NTP sources for redundancy |
| Log retention — online | Logs must be searchable for a minimum of **90 days** on the syslog collector |
| Log retention — archive | Logs must be archived for a minimum of **12 months** in cold storage or a SIEM |

---

## Login Warning Banner

A legal warning banner must be displayed before authentication on all management interfaces (SSH, serial console, and web). This is required by ISO 27001 A.5.14, NIST SP 800-53 AC-8, and PCI-DSS Requirement 8.

The banner must:

- State that the system is for authorized users only
- Warn that activity is monitored and logged
- Not disclose system-specific information (e.g. OS version, hostname) to unauthenticated users

**Recommended banner text:**

```
*******************************************************************************
*                        AUTHORIZED ACCESS ONLY                               *
*                                                                              *
*  This system is the property of [Organization Name] and is for authorized   *
*  use only. Unauthorized or improper use of this system may result in        *
*  administrative disciplinary action and/or civil and criminal penalties.    *
*                                                                              *
*  By accessing and using this system, you consent to monitoring and          *
*  recording of all activity. Evidence of unauthorized or malicious use may   *
*  be referred to law enforcement or other relevant authorities.              *
*                                                                              *
*  Disconnect IMMEDIATELY if you are not an authorized user.                  *
*******************************************************************************
```

---

## Vulnerability and Patch Management

RansNet monitors the relevant system component CVE feeds and issues firmware updates according to the patch timelines below:

| Severity | CVSS Score | Maximum Patch Timeline |
|---|---|---|
| **Critical** | 9.0–10.0 | 14 days from disclosure |
| **High** | 7.0–8.9 | 30 days from disclosure |
| **Medium** | 4.0–6.9 | Next scheduled firmware release |
| **Low / Informational** | 0.1–3.9 | Best-effort; addressed in major release |

!!! note
    Patch timelines above represent the maximum target for a firmware release to be available. Deployment to production devices is subject to the customer's own change management process.

When a critical or high severity CVE affects a deployed component, RansNet will:

1. Publish a security advisory in the release notes
2. Release a patched firmware build within the timeline above
3. Notify affected customers via the support portal

---

## Compliance Mapping

The table below maps RansNet hardening controls to commonly referenced compliance frameworks to assist auditors and integrators.

| Control Area | CIS Controls v8 | ISO 27001:2022 | NIST SP 800-53 Rev 5 |
|---|---|---|---|
| Default credentials changed | CIS 4.7 | A.5.17 | IA-5 |
| Password complexity and lockout | CIS 5.2, 5.3 | A.5.17 | IA-5, IA-11 |
| SSH root login disabled | CIS 4.1 | A.8.2 | AC-2, AC-6 |
| Login warning banner | CIS 4.1 | A.5.14 | AC-8 |
| Management ACLs / source IP restriction | CIS 12.2 | A.8.20 | AC-3, SC-7 |
| SNMP v3 (if SNMP used) | CIS 12.1 | A.8.20 | AC-3 |
| Unused services disabled | CIS 4.8 | A.8.8 | CM-7 |
| Stateful firewall, default deny | CIS 13.1 | A.8.20 | SC-7 |
| TLS 1.2/1.3, strong ciphers | CIS 3.10 | A.8.24 | SC-8, SC-28 |
| VPN / IPSec strong algorithms | CIS 3.10 | A.8.24 | SC-8 |
| System and security logging | CIS 8.2 | A.8.15 | AU-2, AU-3 |
| Config change audit trail | CIS 8.5 | A.8.15 | AU-12, CM-5 |
| NTP synchronization | CIS 8.4 | A.8.17 | AU-8 |
| Log retention (90 days online, 12 months archive) | CIS 8.3 | A.8.15 | AU-11 |
| Firmware update process | CIS 7.3, 7.4 | A.8.8 | SI-2 |
| CVE patch timelines | CIS 7.4 | A.8.8 | SI-2, RA-5 |

---

## Compliance and Evidence

To demonstrate compliance with this baseline during audits, collect and retain the following evidence artefacts:

| Evidence | Description |
|---|---|
| **Firewall configuration export** | Full firewall ruleset showing default deny, management ACLs, and rate limiting |
| **Login banner screenshot** | Screenshot of SSH/console login showing the warning banner before authentication |
| **User account list** | Output of configured accounts with roles; confirm no default/shared accounts remain |
| **SNMP configuration** | Confirm SNMP v1/v2c disabled; show SNMPv3 configuration if in use |
| **Syslog configuration** | Show remote syslog destinations configured and log severity settings |
| **Log sample** | Sample of authentication, firewall drop, and config change events from the syslog collector |
| **NTP configuration** | Show NTP server configuration and synchronisation status |
| **Firmware version** | Show current firmware version and confirm it is within the supported release window |
| **Signed onboarding checklist** | Completed checklist (below) signed by the implementing engineer and project lead |

This approach aligns with CIS Benchmarks where applicable and documents deviations where the embedded operating system or device role requires a different implementation. Deviations must be recorded with a compensating control and approved through the RansNet security governance process.

---

## Onboarding Hardening Checklist

Use this checklist as part of your UAT/implementation report. Each item must be verified by the implementing engineer and signed off before the device enters production.

**Device:** ___________________________  **Serial / ID:** ___________________________

**Engineer:** ___________________________  **Date:** ___________________________

**Reviewed by:** ___________________________  **Date:** ___________________________

---

**OS and Services**

- [ ] Default login password changed to a unique, policy-compliant password
- [ ] Enable/privileged password set separately from login password
- [ ] Login warning banner configured on SSH and console
- [ ] Unused services confirmed disabled (telnet, FTP)
- [ ] Firmware version confirmed as latest STABLE release

**Management and Access**

- [ ] Management interface assigned to dedicated VRF or restricted interface
- [ ] Firewall-input rules permit only authorized source IPs for SSH/HTTPS
- [ ] SSH root login disabled (`PermitRootLogin no`)
- [ ] SSH `MaxAuthTries` set to 3
- [ ] SSH idle timeout configured (10 minutes)
- [ ] SNMP v1/v2c confirmed disabled; SNMPv3 configured if monitoring required

**Firewall and Network**

- [ ] Stateful firewall enabled
- [ ] Default deny policy in place on WAN/untrusted interfaces
- [ ] No "permit all" rules present
- [ ] Rate limiting configured for SSH, HTTPS, and ICMP
- [ ] Unused physical ports disabled

**Cryptography**

- [ ] TLS 1.2/1.3 only (no SSL, TLS 1.0, TLS 1.1)
- [ ] VPN/IPSec policy uses IKEv2, AES-256, SHA-256, DH group ≥ 14
- [ ] SSL certificates are valid and will not expire within 90 days

**Logging and Monitoring**

- [ ] Syslog remote destination configured and reachable
- [ ] Firewall rule logging enabled
- [ ] Authentication and config change logging confirmed (test log entry generated)
- [ ] NTP synchronization configured with at least one reachable server
- [ ] NTP authentication configured (if supported by platform)

**Documentation**

- [ ] Configuration export archived in document management system
- [ ] Baseline configuration backup taken
- [ ] Deviations from this baseline documented with compensating controls

---

## Review and Updates

This hardening baseline is reviewed at least annually or whenever significant security requirements change. Any deviations are documented and approved through the RansNet security governance process.

| Field | Value |
|---|---|
| **Document owner** | RansNet Security Engineering |
| **Review cycle** | Annual, or upon significant CVE disclosure |
| **Last reviewed** | April 2026 |
| **Next review due** | April 2027 |
