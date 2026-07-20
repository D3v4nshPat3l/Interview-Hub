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

## 1. What is Active Directory Domain Services?

**Level:** Beginner

**Answer:** Active Directory Domain Services, or AD DS, is the Windows Server directory service used to store and manage information about identities and resources in a domain. Its directory contains objects such as users, computers, groups, managed service accounts, organizational units, Group Policy containers, and configuration data. AD DS provides centralized authentication, authorization support, policy distribution, directory search, and replication between domain controllers. It uses several protocols rather than one protocol: DNS locates services, Kerberos is the preferred domain authentication protocol, LDAP exposes directory operations, SMB provides access to SYSVOL and NETLOGON, and RPC is used for many management and replication functions. A strong interview answer distinguishes the product from the broader term “Active Directory,” which can also refer to technologies such as AD CS, AD FS, and AD LDS.

## 2. What business problems does AD DS solve?

**Level:** Beginner

**Answer:** AD DS provides a common identity and policy plane for a Windows enterprise. Instead of creating separate accounts and permissions on every server, administrators create identities centrally, place them in groups, and grant those groups access to resources. Users can authenticate once and access authorized services without maintaining an unrelated password for each system. Administrators can deploy security settings, certificates, scripts, software, drive mappings, and operating-system configuration through Group Policy. AD DS also supports delegated administration, allowing a help-desk team to reset passwords in a specific OU without making its members Domain Admins. Operationally, it improves consistency, scalability, auditing, and recovery. The security tradeoff is concentration of trust: compromise of highly privileged AD identities or domain controllers can affect most systems that depend on the directory.

## 3. What is an Active Directory object?

**Level:** Beginner

**Answer:** An AD object is an instance of a class defined by the directory schema. A user, computer, group, OU, site, subnet, GPO container, and service account are all examples. Each object has attributes that describe it, such as `sAMAccountName`, `userPrincipalName`, `objectGUID`, `objectSid`, `member`, or `dNSHostName`. Objects are identified and located through names such as a distinguished name, but security references normally use immutable or relatively stable identifiers such as a SID or GUID. Some objects are security principals and can be assigned permissions; others are containers or configuration objects. In an interview, avoid saying that every object is a file or that every object can authenticate. The schema determines what attributes an object may have, while access control determines who may read or modify them.

## 4. What is the difference between authentication and authorization in AD?

**Level:** Beginner

**Answer:** Authentication establishes the identity of a user, computer, or service. In an AD domain, this commonly occurs through Kerberos and may fall back to NTLM in specific circumstances. Authorization determines what the authenticated security principal is permitted to do. Windows builds an access token containing the principal's SID, relevant group SIDs, privileges, integrity information, and other claims. When the principal accesses an object such as a file, registry key, service, or AD object, the system compares that token with the object's security descriptor and access control entries. Successful authentication therefore does not imply access. A user may prove identity correctly but still be denied because the required permission is absent, an explicit deny applies, a policy restriction exists, or the resource is unavailable.

## 5. What is an AD domain?

**Level:** Beginner

**Answer:** An AD domain is a logical partition of a forest with its own domain naming context, DNS name, domain SID, security policies, domain-level groups, and domain controllers. All writable domain controllers in the domain hold a writable replica of that domain partition. A domain is an administrative and replication boundary for much directory data, but it is not the strongest security isolation boundary inside a forest. Enterprise Admins, forest-wide configuration, schema control, trusts, and several escalation paths mean that domains in the same forest should be treated as part of one security boundary. Domains are usually created because of namespace, replication, legal, administrative autonomy, or historical requirements - not merely because the organization has multiple departments.

## 6. What is an organizational unit, and why is it used?

**Level:** Beginner

**Answer:** An organizational unit, or OU, is a container within a domain used primarily for delegation and Group Policy scope. Administrators can create an OU structure that separates workstations, servers, users, service accounts, and administrative responsibilities. An OU is not a security boundary: moving an object into an OU does not by itself isolate it from domain administrators or forest-level control. OUs should be designed around management requirements rather than copied blindly from the company organization chart. A useful design asks which objects require different GPOs, which teams need delegated control, and which lifecycle processes apply. Excessive nesting makes policy inheritance and delegation difficult to reason about, while placing all objects in default containers limits policy and delegation flexibility.

## 7. What is a domain tree?

**Level:** Beginner

**Answer:** A domain tree is one or more AD domains that share a contiguous DNS namespace. For example, `corp.example.com`, `emea.corp.example.com`, and `lab.emea.corp.example.com` are in one namespace tree. Parent-child domains created within a forest receive automatic two-way transitive trusts. A tree describes namespace relationships; it does not mean the entire forest has only one domain tree. Multiple trees with different DNS roots can exist in the same forest while sharing the forest schema, configuration partition, and global catalog. Modern designs usually prefer fewer domains because additional domains create more domain controllers, DNS and replication complexity, and administrative overhead.

## 8. What is an AD forest?

**Level:** Beginner

**Answer:** A forest is the top-level AD DS logical structure and the primary security boundary. It contains one or more domains that share a common schema, configuration partition, and global catalog. The first domain created is the forest root domain, and its DNS name also identifies the forest. Domains in the same forest are connected by automatic transitive trusts and are not considered mutually isolated against forest-level administrators or a sufficiently capable attacker. Decisions that affect the schema, domain naming, forest trusts, and certain enterprise-wide roles occur at forest scope. Creating a separate forest is justified when a genuinely separate security boundary, schema ownership model, or independent recovery authority is required, but cross-forest collaboration then needs explicit trust, synchronization, or application integration.

## 9. Why is the forest considered the AD security boundary?

**Level:** Intermediate

**Answer:** A domain can delegate administration and contain its own domain data, but forest-level control can ultimately influence every domain in that forest. Enterprise Admins can administer domains across the forest, Schema Admins can change the schema, and the configuration partition is forest-wide. Trust relationships and Kerberos behavior also connect domains. In addition, compromise of a domain controller, privileged replication rights, or forest-level administrative path can enable escalation beyond one domain. Therefore, two business units that do not trust each other's administrators should not be placed in the same forest merely for convenience. A forest is not an absolute boundary against hypervisor, backup, PKI, identity-sync, or physical administrators; the complete control plane must also be included in the threat model.

## 10. What is the difference between a logical and physical AD structure?

**Level:** Beginner

**Answer:** The logical structure consists of forests, trees, domains, OUs, objects, and directory partitions. It describes namespace, delegation, policy scope, and the organization of directory data. The physical structure consists mainly of domain controllers, sites, subnets, site links, replication connections, and the underlying network. A user may belong to an OU based on management needs while the user's workstation is mapped to a site based on its IP subnet. The two structures solve different problems: OUs do not control replication topology, and sites do not provide an OU-like administrative hierarchy. Confusing sites with domains or OUs is a common interview mistake.

## 11. What is an Active Directory site?

**Level:** Beginner

**Answer:** An AD site represents one or more well-connected IP subnets. Sites influence domain-controller location, replication topology, service affinity, and traffic optimization. Clients determine their site from their IP address and subnet mappings, then DC Locator prefers an appropriate domain controller in or near that site. Intersite replication is scheduled and cost-aware, whereas intrasite replication is optimized for fast, reliable networks. A site is not part of the DNS namespace and is not a security boundary. Accurate subnet-to-site mapping is critical; missing or incorrect subnets can send clients to remote domain controllers, increase WAN traffic, slow sign-in, and create misleading troubleshooting symptoms.

## 12. What is a directory partition or naming context?

**Level:** Intermediate

**Answer:** A directory partition, also called a naming context, is a replicated portion of the AD database with a defined replication scope. The domain partition contains domain-specific objects such as users, computers, and groups and replicates to domain controllers in that domain. The configuration partition stores forest-wide topology and service configuration and replicates to every domain controller in the forest. The schema partition defines object classes and attributes and also replicates forest-wide. Application partitions can hold application-specific data with a chosen replica set; AD-integrated DNS commonly uses `DomainDnsZones` and `ForestDnsZones`. Understanding partitions explains why an LDAP search base matters and why not every domain controller stores every object as a full writable replica.

## 13. What is the Active Directory schema?

**Level:** Intermediate

**Answer:** The schema is the forest-wide definition of the object classes and attributes that may exist in AD DS. It defines, for example, which attributes a user object can contain, the syntax of those attributes, and whether attributes are indexed or included in special sets. Schema changes replicate to all domain controllers and are generally difficult to reverse, so extensions require testing, change control, backups, vendor validation, and a rollback strategy based on disabling or superseding rather than deleting definitions. Only the Schema Master accepts schema modifications. Applications such as Exchange may extend the schema. A schema extension does not automatically grant users access to new attributes; normal ACLs still apply.

## 14. What are domain and forest functional levels?

**Level:** Intermediate

**Answer:** Functional levels enable AD DS capabilities that require every domain controller in the relevant scope to support them. A domain functional level affects domain features and constrains the operating systems that can run as domain controllers in that domain. A forest functional level affects forest-wide capabilities and requires compatible levels across domains. Functional level is not the same as the Windows Server version on member servers or clients. Current interviews may include Windows Server 2025 functional levels, but an administrator should first inventory domain-controller versions, application dependencies, replication health, and backup readiness. Raising a level should be a controlled change, not a routine click performed simply because newer servers exist.

## 15. What is the difference between AD DS and Microsoft Entra ID?

**Level:** Beginner

**Answer:** AD DS is a domain directory designed around Windows enterprise networks, domain controllers, Kerberos, NTLM, LDAP, Group Policy, domain join, and hierarchical objects. Microsoft Entra ID is a cloud identity and access platform designed for modern application authentication and authorization using protocols such as OAuth 2.0, OpenID Connect, and SAML. Entra ID has tenants, users, groups, applications, service principals, roles, Conditional Access, and cloud device identities, but it does not provide traditional AD domain controllers or classic GPO processing. Hybrid organizations synchronize or provision selected identities between the systems, but synchronization does not make them one database. Microsoft Entra Domain Services is a separate managed-domain service that exposes LDAP, Kerberos, NTLM, and Group Policy for compatible workloads.

## 16. What is AD LDS, and how is it different from AD DS?

**Level:** Intermediate

**Answer:** Active Directory Lightweight Directory Services is an LDAP directory service that can run multiple application-specific directory instances without creating a Windows security domain. It uses the same underlying directory technology and many AD-compatible APIs, but it does not provide domain join, domain user logon, Group Policy, or domain controllers in the AD DS sense. Applications can define an appropriate schema and store directory data without extending the enterprise AD DS forest. AD LDS is useful when an application needs LDAP and directory semantics but should not depend on or modify the corporate identity directory.

## 17. What is AD CS?

**Level:** Beginner

**Answer:** Active Directory Certificate Services is Microsoft's public key infrastructure role for issuing, managing, validating, renewing, and revoking digital certificates. In an enterprise deployment, AD CS can integrate with AD DS for certificate templates, autoenrollment, access control, and identity mapping. Certificates may support TLS, user or computer authentication, smart-card sign-in, code signing, IPsec, and other cryptographic use cases. AD CS is security-critical because a certificate that maps to a privileged identity can be equivalent to that identity's authentication capability. Interview answers should therefore include governance of certification authorities, offline roots, template permissions, issuance requirements, revocation, key protection, auditing, and recovery - not only the definition of a CA.

## 18. What is AD FS?

**Level:** Beginner

**Answer:** Active Directory Federation Services is a federation service that issues security tokens so users can access claims-aware applications across organizational or security boundaries. AD FS typically authenticates a user against AD DS and then issues a signed token containing claims for a relying party. It has historically been used for federated sign-in to Microsoft 365 and third-party applications. Many organizations now prefer cloud authentication with Microsoft Entra ID because AD FS introduces additional servers, certificates, proxy components, monitoring, and recovery requirements. Where AD FS remains, its signing keys, service account, configuration database, and administrative plane are Tier 0 assets.

## 19. What is single sign-on in an AD environment?

**Level:** Beginner

**Answer:** Single sign-on means a user authenticates once and can subsequently access multiple authorized resources without repeatedly entering credentials. In an AD domain, Windows integrated authentication commonly uses the user's logon session and Kerberos tickets to achieve SSO to services with correctly registered SPNs. SSO is not the same as using one password everywhere by manual re-entry, and it does not bypass authorization. Failures often occur because of SPN problems, DNS aliases, time skew, browser zone settings, delegation requirements, or fallback to NTLM. In hybrid environments, seamless sign-on to cloud applications may involve Entra join, hybrid join, Primary Refresh Tokens, federation, or Entra Connect features rather than classic domain Kerberos alone.

## 20. What are the most important protocols and ports associated with AD DS?

**Level:** Intermediate

**Answer:** Important services include DNS on TCP/UDP 53, Kerberos on TCP/UDP 88, LDAP on TCP/UDP 389, LDAPS on TCP 636, Global Catalog LDAP on TCP 3268 and 3269, SMB on TCP 445, RPC Endpoint Mapper on TCP 135, dynamic RPC ports, Kerberos password change on TCP/UDP 464, and NTP on UDP 123. ICMP can also matter for diagnostics and path behavior. Port lists alone are insufficient: the communication direction, dynamic RPC range, site topology, firewall state, name resolution, authentication method, and certificate requirements determine whether a workflow succeeds. Avoid the common mistake of opening only 389 and 88 between sites and assuming all domain operations will work.

---
# Objects, Schema, Naming, Groups, and Delegation

*Identity attributes, security principals, naming, access control, and delegated administration.*

## 21. What is the difference between a security principal and a non-security-principal object?

**Level:** Beginner

**Answer:** A security principal is an identity that can be authenticated or represented in an access token and assigned permissions. Users, computers, managed service accounts, and security groups are common examples. They have a SID, and ACLs can grant or deny them rights. A distribution group is an AD object but is not security-enabled and therefore is not used to authorize access. OUs, contacts, sites, and most configuration objects are also not security principals. The distinction matters because placing an object in AD does not automatically make it capable of signing in or receiving permissions. When troubleshooting access, confirm that the group is security-enabled, the expected SID is present in the user's token, and token refresh has occurred after membership changes.

## 22. What is a Security Identifier?

**Level:** Beginner

**Answer:** A SID is the identifier Windows uses to represent a security principal in access control. A domain account SID contains the domain SID plus a relative identifier, or RID, that uniquely identifies the object within the domain. ACLs generally store SIDs rather than account names, which is why renaming a user does not remove existing permissions. Deleting and recreating a user with the same name produces a different SID, so the new object does not automatically inherit the old object's ACL access. `SIDHistory` can preserve authorization during migration, but it is sensitive and must be protected, audited, and removed when no longer required. Well-known SIDs represent built-in identities such as Local System or Authenticated Users.

## 23. What is the difference between `objectGUID`, SID, and distinguished name?

**Level:** Intermediate

**Answer:** `objectGUID` is a globally unique identifier assigned when an AD object is created and normally remains unchanged when the object is renamed or moved within the forest. A SID identifies a security principal for authorization; not every object has one. A distinguished name, or DN, represents the object's current LDAP path, such as `CN=Alice,OU=Users,DC=corp,DC=example,DC=com`, and changes when the object is renamed or moved. Applications should not use a DN as an immutable identity key. During migrations or synchronization, engineers must understand which identifier a product anchors to, because matching on names can create duplicates or accidental joins.

## 24. What is a distinguished name, relative distinguished name, and canonical name?

**Level:** Beginner

**Answer:** A distinguished name is the full LDAP path of an object from the object to the directory root. The relative distinguished name, or RDN, is the first component that names the object within its parent, such as `CN=Alice`. A canonical name is a more human-readable path such as `corp.example.com/Users/Alice`. LDAP tools typically use DNs for search bases and modifications, while administrative interfaces may display canonical names. Special characters in DNs must be escaped correctly. A user's sign-in name is not necessarily the same as the CN or DN, so changing the display name or moving the object does not automatically change the UPN or `sAMAccountName`.

## 25. What is the difference between `sAMAccountName` and UPN?

**Level:** Beginner

**Answer:** `sAMAccountName` is the legacy account name used in forms such as `DOMAIN\username` and is limited by older compatibility rules. A user principal name, or UPN, has the form `user@suffix` and is the preferred user-friendly sign-in name for many modern applications. The UPN suffix does not have to match the AD domain DNS name if an alternative suffix is configured in the forest. A UPN is not an email address by definition, although organizations often align them. Duplicate or unverified suffixes can cause hybrid identity and application problems. During a domain or tenant migration, UPN planning is a key dependency because it affects user experience, synchronization matching, and cloud sign-in.

