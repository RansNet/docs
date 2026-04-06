# Device Hardening

This page describes the CPE security hardening baseline that RansNet applies to Customer Premises Equipment (CPE) such as SD-WAN gateways (CMG, HSA, UA) to comply with industry best practices, including CIS Benchmarks where applicable, during device onboarding, provisioning, and firmware updates — ensuring devices are securely configured before deployment.

---

## What are CIS Benchmarks?

CIS Benchmarks are security hardening standards developed by the Center for Internet Security (CIS). They outline detailed, widely accepted configuration recommendations for systems ranging from operating systems (e.g. Linux, Windows) to network devices and applications.

For embedded CPE devices, not all CIS controls apply directly. RansNet CPE device OS/image are pre-built with functionally equivalent controls where native CIS rules cannot be applied due to platform constraints. These benchmarks serve as a baseline for secure configuration that auditors, integrators, and service providers often reference.

RansNet CPE hardening aligns with the following CIS Benchmarks:

- **Primary:** CIS Benchmark for Linux (Level 1 – Enterprise) — sections applicable to embedded systems
- **Supplementary:** CIS Benchmark for Network Devices (v2.0)
- **Framework Reference:** CIS Critical Security Controls v8

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
| Unused services (e.g. telnet, FTP) are disabled | OS default |
| Privileged access (e.g. root/enable) is restricted | Ensure login and enable passwords are different and comply with corporate security policy |
| Firmware is kept up-to-date | Upgrade to latest STABLE firmware after onboarding |

### 2. Management and Access Controls

| Control | Action Required |
|---------|----------------|
| Management access (SSH/HTTPS) enabled only over secure channels | Configure VRF for management interface; add firewall-input rules to permit authorized source IPs only |
| Telnet and unencrypted management interfaces are disabled | OS default |
| Idle session timeouts are configured | OS default |
| SSH key-based authentication enforced where practical | Recommended: disable SSH, or configure firewall-input rules to allow authorized source IPs only |

### 3. Network and Firewall Hardening

| Control | Action Required |
|---------|----------------|
| Stateful firewall is enabled | OS default |
| Default deny rules apply to untrusted interfaces | OS default |
| Management and control ports restricted by ACLs | Configure VRF for management interface; add firewall-input rules to permit authorized source IPs only |
| Address translation and NAT used responsibly | Review firewall rules — strictly no "permit all" type rules |
| Rate limiting for TCP SYN, ICMP, and other flood vectors | Configure rate limiting for management (SSH/HTTPS) services if these are allowed |

### 4. Cryptography and TLS

| Control | Action Required |
|---------|----------------|
| Only TLS 1.2 and TLS 1.3 are enabled | OS default |
| Weak protocols (SSL, TLS 1.0, TLS 1.1) are disabled | OS default |
| Cipher suites adhere to strong cryptographic practices (forward secrecy, AEAD) | OS default |
| Certificate validation follows industry best practices | Review company SOP; use valid SSL certificates and rotate yearly |

### 5. Logging and Monitoring

| Control | Action Required |
|---------|----------------|
| System and security logging is enabled | Enable logging for firewall rules; export logs to a syslog collector |
| Authentication attempts, firewall drops, and config changes are logged | Deploy syslog collector to receive logs from all devices |
| NTP time synchronization configured for consistent timestamps | Use customer's internal NTP server if available |
| Logs retained according to operational procedures | Configure log archival for long-term retention |

---

## Compliance and Evidence

To demonstrate compliance with this baseline during audits:

- Provide configuration screenshots or exports of firewall rules.
- Include logs showing management ACLs, rate limiting, and disabled services.
- Attach a signed device onboarding checklist confirming that hardening controls have been verified.

This approach aligns with CIS Benchmarks where applicable and documents deviations where the embedded operating system or device role requires a different implementation.

---

## Onboarding Hardening Checklist

Use this checklist as part of your UAT/implementation report:

- Default passwords changed
- Unused services disabled
- Firewall enabled and default deny in place
- Management access restricted (SSH/HTTPS)
- Rate limiting enabled for SYN, ICMP, UDP
- TLS configuration restricted to TLS 1.2/1.3 with strong ciphers
- Logging enabled and verified
- Configuration export archived

---

## Review and Updates

This hardening baseline is reviewed at least annually or whenever significant security requirements change. Any deviations are documented and approved through the RansNet security governance process.
