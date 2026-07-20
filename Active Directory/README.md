# Active Directory Interview Questions and Answers

> A repo-ready Active Directory Domain Services interview guide with **300 technically validated questions and detailed answers**, progressing from fundamentals to L2/L3 administration, architecture, security engineering, incident response, hybrid identity, and senior scenario-based troubleshooting.

This README is designed for `Interview-Hub/Active Directory`. It consolidates the supplied interview lists and PDFs, then rewrites and expands them into an original study guide grounded primarily in current Microsoft documentation. The final 100 questions are realistic operational, security, migration, and disaster-recovery scenarios phrased in the style commonly used in technical interviews.

> **Scope:** The primary subject is on-premises **Active Directory Domain Services (AD DS)**. Microsoft Entra ID, Microsoft Entra Connect, AD FS, AD CS, Windows LAPS, managed service accounts, and Defender for Identity are included where they intersect with AD DS.
>
> **Security note:** Attack-path questions explain prerequisites, impact, detection, containment, and remediation. They are not exploitation runbooks. Use all security techniques only in systems for which you have explicit authorization.

---

## How to Use This Guide

- Use **Questions 1-60** to build precise AD DS architecture and identity fundamentals.
- Use **Questions 61-100** for DNS, DC Locator, Kerberos, NTLM, SPNs, and delegation.
- Use **Questions 101-160** for Group Policy, SYSVOL, replication, sites, trusts, FSMO, backup, and recovery.
- Use **Questions 161-200** for hybrid identity, AD CS, LAPS, service accounts, RODCs, attacks, detection, and hardening.
- Use **Questions 201-300** as mock L2/L3, security, architecture, migration, and incident-response interviews.

For each question, practice a three-layer response:

1. **State the definition or decision clearly.**
2. **Explain the mechanism and operational impact.**
3. **Give validation steps, limitations, and the next diagnostic action.**

---

## Source and Validation Method

The supplied interview pages and PDFs were used to identify recurring interview themes. Technical statements were then checked against current Microsoft Learn documentation, Windows protocol specifications, RFC 4120, and MITRE ATT&CK where applicable.

Several source-quality corrections were deliberately applied:

- `SYSVOL` stores Group Policy files and sign-in scripts; it does **not** store the AD database or its transaction logs. The AD database is normally `%SystemRoot%\NTDS\ntds.dit`.
- A Global Catalog holds a writable full replica of its own domain plus a read-only partial attribute set for objects in other forest domains; it is not automatically present on every domain controller.
- Current domains should use DFS Replication for SYSVOL. FRS, REPLMON, and many Windows 2000-era procedures are historical or migration-only material.
- Tombstone lifetime is not safely memorized as one universal number. Check the forest's configured value and creation history.
- Microsoft Entra ID is not simply “Active Directory in Azure.” It is a cloud identity platform with a different architecture and protocol model from AD DS.
- “Enable MFA in Active Directory” requires architectural detail. AD DS alone does not expose a universal MFA switch for every Kerberos or NTLM sign-in.

The diagrams in the supplied administration PDF were used to preserve the correct learning progression from OU and domain to tree, forest, and site. The trust-direction diagrams in the offensive reference were used to frame interview explanations around the critical rule that **trust direction and access direction are opposite**.

---

## Contents

1. [Foundations and Logical Architecture](#foundations-and-logical-architecture) - Questions 1-20
2. [Objects, Schema, Naming, Groups, and Delegation](#objects-schema-naming-groups-and-delegation) - Questions 21-40
3. [Domain Controllers, Database, Global Catalog, and FSMO](#domain-controllers-database-global-catalog-and-fsmo) - Questions 41-60
4. [DNS, DC Locator, Time, Ports, and Secure Channels](#dns-dc-locator-time-ports-and-secure-channels) - Questions 61-80
5. [Kerberos, NTLM, SPNs, and Delegation](#kerberos-ntlm-spns-and-delegation) - Questions 81-100
6. [Group Policy and SYSVOL](#group-policy-and-sysvol) - Questions 101-120
7. [Replication, Sites, Trusts, Backup, and Recovery](#replication-sites-trusts-backup-and-recovery) - Questions 121-160
8. [Hybrid Identity, AD CS, LAPS, Service Accounts, and RODCs](#hybrid-identity-ad-cs-laps-service-accounts-and-rodcs) - Questions 161-180
9. [AD Security, Attack Paths, Detection, and Hardening](#ad-security-attack-paths-detection-and-hardening) - Questions 181-200
10. [L2 Operations Scenarios](#l2-operations-scenarios) - Questions 201-220
11. [L3 DNS, Replication, Authentication, and GPO Scenarios](#l3-dns-replication-authentication-and-gpo-scenarios) - Questions 221-240
12. [Security and Incident-Response Scenarios](#security-and-incident-response-scenarios) - Questions 241-260
13. [Architecture, Migration, and Hybrid Scenarios](#architecture-migration-and-hybrid-scenarios) - Questions 261-280
14. [Senior Engineering and Disaster-Recovery Scenarios](#senior-engineering-and-disaster-recovery-scenarios) - Questions 281-300
15. [Command and Event Reference](#command-and-event-reference)
16. [Primary References](#primary-references)

---

# Foundations and Logical Architecture

*Core definitions, boundaries, hierarchy, and design language.*

