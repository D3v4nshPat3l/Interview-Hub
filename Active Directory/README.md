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

## 26. What is `userAccountControl`?

**Level:** Intermediate

**Answer:** `userAccountControl` is a bitmask attribute containing multiple account-control flags. Examples include account disabled, password never expires, smart card required, trusted for delegation, do not require Kerberos preauthentication, and normal account. Because several flags can be combined, the decimal value should not be interpreted as one simple state. PowerShell and administrative tools normally expose friendly properties, but LDAP filters and forensic investigations may inspect the bitmask directly. Security reviews should identify risky flags such as reversible password encryption, unnecessary delegation, disabled preauthentication, or nonexpiring passwords. A change to one flag can have authentication and attack-surface consequences beyond the visible checkbox.

## 27. What is an SPN, and which objects can hold one?

**Level:** Intermediate

**Answer:** A Service Principal Name identifies a service instance to Kerberos, normally in a form such as `HTTP/web01.corp.example.com`. The KDC uses the SPN to determine which account's key should protect the service ticket. SPNs are stored on the account under which the service runs, commonly a computer account, user-based service account, gMSA, or dMSA. An SPN should be unique in the forest. Duplicate or missing SPNs can cause Kerberos failure, `KRB_AP_ERR_MODIFIED`, or NTLM fallback. Administrators should use `setspn -S` rather than blindly adding values because `-S` checks for duplicates. DNS aliases, load balancers, service migrations, and account changes frequently require deliberate SPN updates.

## 28. What are group type and group scope?

**Level:** Beginner

**Answer:** Group type determines whether a group is security-enabled or used only for distribution. Group scope determines where members may come from and where the group may be used. Domain Local groups are normally used to assign permissions to resources in their own domain and can contain members from trusted domains. Global groups normally collect accounts from their own domain and can be granted permissions in trusting domains. Universal groups can contain members from across the forest and can be used forest-wide or across trusts. Universal membership is stored in the Global Catalog, so frequent changes in large multi-domain forests can increase replication traffic. Correct group design separates role membership from resource permission assignment.

## 29. Explain AGDLP and AGUDLP.

**Level:** Intermediate

**Answer:** AGDLP means Accounts are placed into Global groups, Global groups are placed into Domain Local groups, and permissions are assigned to the Domain Local groups. This separates business-role membership from resource ACLs and makes access easier to review. AGUDLP inserts Universal groups between Global and Domain Local groups for multi-domain designs: Accounts -> Global -> Universal -> Domain Local -> Permissions. These are design patterns, not enforced technical rules. The best implementation uses clear naming, avoids deep or circular nesting, documents owners, and periodically reviews effective membership. Directly assigning individual users to file ACLs may work initially but creates audit and lifecycle problems.

## 30. What is group nesting, and what are its risks?

**Level:** Intermediate

**Answer:** Group nesting places one group inside another so access can be modeled through roles and resource groups. It reduces repetitive administration but can obscure effective access when nesting is deep, cross-domain, circular, or undocumented. Token size can grow when users are members of many groups, potentially contributing to Kerberos or HTTP header problems. Universal group changes can create forest-wide catalog replication. Privileged groups hidden several levels deep are easy to miss during review. A mature design limits nesting depth, prohibits circular membership, uses ownership metadata, monitors changes to sensitive groups, and regularly resolves transitive membership rather than reviewing only direct members.

## 31. What is an ACL, ACE, and security descriptor in Active Directory?

**Level:** Intermediate

**Answer:** A security descriptor defines the owner, discretionary access control list, system access control list, and control flags for a securable object. The DACL contains access control entries that allow or deny specific rights to trustees identified by SID. The SACL specifies what access should be audited, subject to audit policy. AD ACEs can apply to the whole object, specific attributes, validated writes, extended rights, child object creation, or inherited descendants. Rights such as `GenericAll`, `WriteDACL`, `WriteOwner`, password reset, and replication control rights can be security-critical. Reviewing only group membership is therefore insufficient; delegated ACLs can provide equivalent or indirect privilege.

## 32. What is inheritance in AD permissions?

**Level:** Intermediate

**Answer:** Permission inheritance allows ACEs on a container, domain, or OU to flow to child objects according to inheritance flags and object-class filters. It is the foundation of scalable delegation, but it can also propagate excessive rights. An object can protect its DACL from inheritance, and some privileged objects are periodically protected by AdminSDHolder behavior. Troubleshooting requires identifying whether an ACE is explicit or inherited, the source container, the object types to which it applies, and whether deny entries or protected ACLs are involved. Editing the visible permissions on one object without understanding inheritance may create a temporary fix that is later overwritten or leave similar objects exposed.

## 33. How should password-reset rights be delegated to a help desk?

**Level:** Intermediate

**Answer:** Create a dedicated help-desk security group, place user accounts in a well-defined OU scope, and delegate only the necessary rights - typically reset password, require password change at next sign-in, unlock account, and read limited account properties. Do not grant broad `GenericAll`, Domain Admin membership, or rights over privileged accounts and service accounts. Separate ordinary users from administrators so help-desk delegation cannot reset Tier 0 credentials. Document the delegated ACEs, use separate administrative identities, require MFA at the access layer, log changes, and periodically verify group membership and effective rights. A good answer also mentions that the help desk may need a secure administrative workstation or controlled management endpoint.

## 34. What is the difference between resetting and changing a password?

**Level:** Beginner

**Answer:** A password change is normally initiated by the user and requires knowledge of the current password. A password reset is an administrative action that sets a new password without the old one and therefore requires delegated reset rights. The distinction matters for auditing, credential history behavior, and incident response. Resetting an account after compromise may not invalidate every existing session, ticket, token, cached credential, application password, certificate, or cloud refresh token. Privileged incident response must identify all credential forms and session persistence, not merely perform one AD password reset.

## 35. What is AdminSDHolder?

**Level:** Advanced

**Answer:** AdminSDHolder is a protected object in the System container whose security descriptor is used as a template for certain protected administrative accounts and groups. The SDProp process periodically applies protected ACL behavior, helping prevent delegated OU administrators from controlling highly privileged identities. Protected objects commonly have inheritance disabled and the `adminCount` attribute set. A user removed from a protected group may retain `adminCount=1` and a protected ACL until cleaned up, creating confusing delegation behavior. Security teams should monitor changes to the AdminSDHolder DACL because a malicious ACE can become a persistent forest-wide privilege mechanism for protected accounts.

## 36. What is `SIDHistory`, and when is it used?

**Level:** Advanced

**Answer:** `SIDHistory` stores one or more previous SIDs on a migrated security principal so the new account can continue accessing resources whose ACLs still reference the old SID. It is commonly used during staged domain migrations. Because Windows includes valid SID history values in authorization decisions, unauthorized modification can provide powerful access. Migration tools and trusts must be configured carefully, privileged SID filtering must be understood, and the attribute should be audited. After resource ACLs have been translated and the rollback period has ended, unnecessary SID history should be removed through an approved process. Treat unexpected SID history on privileged or ordinary accounts as a security investigation lead.

## 37. What is an accidental-deletion protection setting?

**Level:** Beginner

**Answer:** Protection from accidental deletion adds deny ACEs that prevent deletion of the object and, where appropriate, deletion of the child from its parent. It is useful for OUs and critical objects because bulk or mistaken deletions are a common operational failure. It is not a backup and does not stop an administrator who deliberately removes the protection or has sufficient control. Combine it with least privilege, change control, Recycle Bin, tested system-state backups, and monitoring of directory-service changes. During automation, scripts should handle this protection explicitly rather than disabling it across a broad scope.

## 38. What is a contact object compared with a user object?

**Level:** Beginner

**Answer:** A contact represents a person or external recipient in the directory but is not a security principal and cannot sign in. It can hold names, email addresses, telephone numbers, and organizational data. A user object can be security-enabled, authenticate, receive group membership, and be assigned permissions. Mail-enabled users and contacts introduce messaging-specific distinctions, but the core AD point is that the existence of an addressable directory entry does not mean it has credentials or a SID. This distinction matters in provisioning, address-list design, and synchronization rules.

## 39. What is the default Computers container, and why do organizations redirect new computer accounts?

**Level:** Intermediate

**Answer:** By default, many newly joined computer accounts are created in `CN=Computers`, which is a container rather than an OU. Traditional GPO links cannot be attached directly to that container, and delegation options are less flexible. Organizations often use `redircmp` or controlled join processes so new computer objects land in a staging OU where baseline policy, inventory, naming checks, and delegated workflows apply. A staging OU should not become a permanent dumping ground; automation should validate and move devices to their managed OUs after enrollment. The same principle applies to the default Users container with `redirusr`, although changes require planning for existing workflows.

## 40. What is the machine-account quota?

**Level:** Advanced

**Answer:** The `ms-DS-MachineAccountQuota` domain attribute controls how many computer accounts an ordinary authenticated user may create in the domain through supported join behavior. Historically, the default is often 10. This can surprise administrators who assume only privileged staff can create machine accounts. In security-sensitive environments, organizations commonly set the quota to zero and delegate computer creation or join rights through controlled OUs and provisioning systems. Changing the quota does not remove previously created computers or fix excessive rights on existing objects. Monitor who creates computer accounts, their attributes, and subsequent delegation or SPN changes.

---

# Domain Controllers, Database, Global Catalog, and FSMO

*Core DC services, directory storage, special roles, and operational placement.*

## 41. What is a domain controller?

**Level:** Beginner

**Answer:** A domain controller is a Windows Server running AD DS that hosts directory partitions and provides domain services such as authentication, LDAP, replication, DC Locator registration, and policy support. Writable domain controllers can originate changes to the domain partition. An RODC receives primarily read-only replicas and uses a controlled password replication policy. Domain controllers are peers in a multi-master system for most operations, but some operations are assigned to FSMO role holders. A DC is not merely a server with the AD Users and Computers console installed. Because domain controllers hold sensitive credentials and trust data, they are Tier 0 systems and should not host unrelated applications or be administered from ordinary workstations.

## 42. Where is the AD database stored?

**Level:** Beginner

**Answer:** The main AD DS database is normally `%SystemRoot%\NTDS\ntds.dit`. It is an Extensible Storage Engine database accompanied by transaction logs and checkpoint files, commonly in the NTDS directory unless paths were changed during promotion. The SYSTEM registry hive contains the boot key material needed as part of credential protection. A consistent domain-controller backup therefore requires AD-aware system-state or supported backup methods rather than copying `ntds.dit` while the service is running. `SYSVOL` is separate and contains policy and sign-in-script files; confusing it with `ntds.dit` is a fundamental error.

## 43. What is stored in `ntds.dit`?

**Level:** Intermediate

**Answer:** `ntds.dit` stores the directory data held by that domain controller, including objects, attributes, link data, schema and configuration replicas, and credential-related values protected by the operating system. A domain controller holds a full replica of its own domain partition; a Global Catalog also holds partial replicas of other domain partitions. The file should never be treated as an ordinary application database. Access to a copy, compatible backup, snapshot, or replication-equivalent credential data can expose domain secrets. Protect domain-controller backups, hypervisor access, storage snapshots, and recovery media with the same rigor as the live DC.

## 44. What is multi-master replication?

**Level:** Intermediate

**Answer:** Multi-master replication means most directory changes can be made on any writable domain controller that hosts the relevant partition. Each DC assigns update sequence numbers locally and replication metadata tracks originating versions, timestamps, and invocation IDs so changes can converge. Multi-master does not mean simultaneous conflicting updates are merged semantically; AD uses conflict-resolution rules. Password changes receive special handling, and some operations are single-master through FSMO roles. The model improves availability and geographic administration, but it depends on DNS, connectivity, time, authentication, and healthy replication topology.

## 45. What is a Global Catalog server?

**Level:** Intermediate

**Answer:** A Global Catalog server is a domain controller that holds a writable full copy of its own domain partition and a read-only partial attribute set for objects in every other domain in the forest. It supports forest-wide searches without referrals and helps evaluate universal group membership during sign-in. The attributes included in the partial attribute set are defined by the schema and selected for common search use. GC queries normally use ports 3268 or 3269. Not every DC must be a GC, although modern environments frequently make all writable DCs GCs unless a specific design constraint exists. GC availability is especially important in multi-domain forests.

## 46. What happens if a Global Catalog is unavailable?

**Level:** Intermediate

**Answer:** Impact depends on forest design, site placement, cached information, and the authenticating account. Forest-wide searches may fail or require referrals. Universal group membership evaluation can affect interactive sign-in in multi-domain environments, although universal group membership caching and specific exceptions may reduce impact. Exchange and other directory-dependent applications may also rely on GC availability. Troubleshoot DNS SRV registration for GC records, client site mapping, network reachability, GC service health, and replication. Do not assume that a server responding on LDAP 389 is also providing Global Catalog service.

## 47. What are FSMO roles, and why do they exist?

**Level:** Beginner

**Answer:** Flexible Single Master Operations roles assign specific operations to one domain controller at a time to avoid conflicts or coordinate tasks that are unsafe or inefficient under pure multi-master behavior. The forest-wide roles are Schema Master and Domain Naming Master. Each domain has a RID Master, PDC Emulator, and Infrastructure Master. The directory can continue functioning for some time when a role holder is offline, but the affected operation eventually fails or operational risk increases. Administrators should know the role function, current holder, dependencies, transfer procedure, seizure criteria, and post-seizure handling.

## 48. What does the Schema Master do?

**Level:** Beginner

**Answer:** The Schema Master is the only domain controller that accepts updates to the forest schema. Schema reads are available from other DCs, and changes then replicate forest-wide. The role is usually lightly loaded and can remain offline temporarily unless an application installation or planned schema change requires it. Before a schema update, validate forest health, back up appropriate domain controllers, review the vendor's LDIF or installation behavior, test in a representative environment, and control membership in Schema Admins. Seizing the role is rare and should follow confirmed permanent loss of the previous holder.

## 49. What does the Domain Naming Master do?

**Level:** Beginner

**Answer:** The Domain Naming Master coordinates adding and removing domains and certain application partitions in the forest. It helps ensure namespace changes remain consistent. Routine authentication does not depend on constant availability of this role, but forest-structure operations do. The role should be reachable and replication-healthy before adding or removing a domain. Administrators should not confuse it with DNS administration; it controls AD forest naming operations, not ordinary creation of DNS host records.

## 50. What does the RID Master do?

**Level:** Intermediate

**Answer:** The RID Master allocates pools of relative identifiers to domain controllers. When a DC creates a security principal, it combines the domain SID with a RID from its local pool to form a unique SID. This allows DCs to create accounts without contacting the RID Master for every object. If the role is unavailable, account creation continues until local pools are exhausted. Monitor RID pool health and investigate unexpectedly high consumption, which may indicate automation errors or abuse. Never attempt to reuse deleted SIDs; uniqueness is central to Windows authorization.

## 51. What does the PDC Emulator do?

**Level:** Intermediate

**Answer:** The PDC Emulator has several high-impact functions. It is the authoritative time source for the domain hierarchy, with the forest-root PDC ultimately synchronized to a reliable external source. It receives urgent password-change information and is consulted when another DC rejects a recently changed password. It is preferred for account lockout processing, Group Policy editing coordination, and certain legacy compatibility functions. Because it affects time, passwords, lockouts, and operations, it should be reliable, well monitored, and placed on capable infrastructure. Moving the role requires validating time configuration afterward.

## 52. What does the Infrastructure Master do?

**Level:** Advanced

**Answer:** The Infrastructure Master maintains references to security principals from other domains, historically updating phantom references when external objects are renamed or moved. In a multi-domain forest where not all DCs are Global Catalogs, placing the Infrastructure Master on a GC can prevent it from seeing stale references because the GC already appears current. If every DC in the domain is a GC, placement does not matter. When Active Directory Recycle Bin is enabled, the role has little or no practical work for cross-domain reference updates. This is a classic interview topic where the correct answer depends on topology rather than a memorized prohibition.

## 53. How do you identify FSMO role holders?

**Level:** Beginner

**Answer:** Common methods include `netdom query fsmo`, `Get-ADForest`, `Get-ADDomain`, and `Get-ADDomainController -Filter * | Select-Object HostName,OperationMasterRoles`. GUI consoles can also show the roles. In an incident or outage, do not stop at identifying the configured holder; verify that the server is alive, advertising, synchronized, and replicating. A stale directory reference or unreachable role holder can exist even though a command prints a name. Document the role distribution as part of routine operations and disaster recovery.

## 54. What is the difference between transferring and seizing an FSMO role?

**Level:** Intermediate

**Answer:** A transfer is a cooperative move performed while the current role holder is online and healthy. The old holder hands off the role cleanly. A seizure forcibly assigns the role when the previous holder is permanently unavailable or cannot be returned safely. Seizure is a recovery action, not a shortcut. After seizing, the old DC should not be brought back online without supported cleanup or rebuilding because two systems believing they own a role can create serious problems. Before seizure, confirm the outage, evaluate how long the operation can wait, check replication, and preserve evidence if compromise is suspected.

## 55. How should FSMO roles be placed?

**Level:** Advanced

**Answer:** Placement should prioritize reliability, replication health, administrative control, and workload rather than arbitrary separation. The Schema Master and Domain Naming Master are often colocated in the forest root. The PDC Emulator should be on a highly available, well-connected DC because of time, password, and lockout functions. The RID Master is commonly colocated with the PDC Emulator. Infrastructure Master placement depends on GC topology and Recycle Bin state. Avoid putting all roles on fragile or remote infrastructure, but do not overengineer separation in a small environment. Document recovery targets and ensure role holders are backed up and monitored.

## 56. What is an RODC?

**Level:** Intermediate

**Answer:** A Read-Only Domain Controller hosts primarily read-only directory replicas and is designed for locations where physical security, connectivity, or local administrative trust is limited. Changes are referred to a writable DC. An RODC can cache selected credentials according to its Password Replication Policy, supports delegated local administration, and uses a separate `krbtgt` account associated with that RODC. It can also use the filtered attribute set to exclude sensitive schema attributes. RODCs reduce some risks but are not disposable appliances; theft or compromise still requires identifying cached credentials, resetting affected passwords, and evaluating trust paths.

## 57. What is the Directory Services Restore Mode password?

**Level:** Intermediate

**Answer:** The DSRM password is a local recovery credential set on a domain controller for offline directory recovery and maintenance. In DSRM, AD DS is not running and authentication relies on the local recovery context rather than normal domain accounts. The password must be unique, protected, rotated, and recoverable under emergency procedures. Windows LAPS can manage and back up DSRM passwords on supported domain controllers. A DSRM credential should not be shared across all DCs or stored in an unsecured runbook. Its use should trigger change-control and security review.

## 58. What is the role of the Netlogon service on a domain controller?

**Level:** Intermediate

**Answer:** Netlogon supports secure-channel operations, domain-controller registration and discovery, authentication-related communication, and publication of DC Locator DNS records. It registers SRV records based on the domain controller's capabilities and site. On clients and members, Netlogon participates in locating a DC and maintaining the machine-account secure channel. Troubleshooting may involve `nltest`, the Netlogon event log, `netlogon.dns`, DNS registration, and secure-channel tests. Restarting Netlogon can trigger DNS record registration, but doing so without fixing DNS or permission problems only treats the symptom.

## 59. What is the difference between promoting a new DC and cloning a DC?

**Level:** Advanced

**Answer:** Promotion installs AD DS and creates a new domain controller through supported replication and configuration workflows. DC cloning is a controlled feature for compatible virtualized domain controllers that uses an authorized source, cloning configuration, and VM-GenerationID-aware safeguards. Copying a DC virtual disk or restoring an arbitrary snapshot is not cloning. A cloned DC receives a new identity and performs supported initialization. Before using cloning, verify hypervisor support, permitted applications, source-DC membership in the cloneable group, network configuration, and recovery planning.

## 60. What is VM-GenerationID, and why is it important for virtualized domain controllers?

**Level:** Advanced

**Answer:** VM-GenerationID is a hypervisor-provided value that allows supported versions of AD DS to detect certain virtual-machine rollback or cloning events. When AD detects a generation change, it can reset its invocation ID and discard the local RID pool, reducing the risk of USN rollback and duplicate SID issuance. It does not make every snapshot workflow safe, replace system-state backup, or protect against copying unsupported images. Hypervisor administrators are effectively part of the Tier 0 trust boundary because they can control domain-controller execution, storage, memory, and network state.

---
# DNS, DC Locator, Time, Ports, and Secure Channels

*The infrastructure dependencies behind sign-in, domain join, authentication, and replication.*

## 61. Why is DNS critical to Active Directory?

**Level:** Beginner

**Answer:** AD DS uses DNS as a service-location system, not merely to translate hostnames to IP addresses. Domain controllers register SRV records for LDAP, Kerberos, Global Catalog, and site-specific roles. Clients query these records through DC Locator to find a suitable domain controller. Replication partners also depend on name resolution. Incorrect client DNS settings can therefore cause domain join failure, slow sign-in, GPO failure, replication errors, and authentication fallback. Domain members should normally use DNS servers that can resolve the AD namespace, usually AD-integrated DNS servers, rather than public resolvers directly. External names can be resolved through forwarders, conditional forwarders, or delegation.

## 62. What DNS records does a domain controller register?

**Level:** Intermediate

**Answer:** A domain controller registers host records and multiple SRV records describing services and capabilities. Common examples include `_ldap._tcp.<domain>`, `_kerberos._tcp.<domain>`, `_ldap._tcp.dc._msdcs.<domain>`, site-specific records under `_sites`, and Global Catalog records for GC servers. The exact set depends on whether the DC is writable, a GC, a KDC, and which site it belongs to. Netlogon performs much of this registration. Use DNS Manager, `nslookup` or `Resolve-DnsName`, and `%SystemRoot%\System32\Config\netlogon.dns` to validate expected records. A record existing is not enough; confirm it points to the correct reachable server and no stale duplicates remain.

## 63. How does DC Locator work?

**Level:** Intermediate

**Answer:** A client calls the local Netlogon service, which uses the DC Locator process to discover and select a domain controller. DNS SRV queries identify candidate DCs, and the client sends a locator request to determine availability and capabilities. Site and subnet information helps prefer a local or closest DC. The result is cached to provide a consistent domain-controller choice. Flags can request a writable DC, GC, PDC, KDC, or other capability. Troubleshooting requires checking the client's DNS servers, domain suffix, subnet mapping, SRV records, site determination, network path, and cached selection. `nltest /dsgetdc:<domain>` and `nltest /dsgetsite` are useful validation tools.

## 64. What is an AD-integrated DNS zone?

**Level:** Intermediate

**Answer:** An AD-integrated zone stores DNS zone data in an AD application or domain partition instead of a traditional primary-zone file. This enables multi-master secure updates and uses AD replication for zone distribution. `DomainDnsZones` normally replicates to DNS servers in a domain, while `ForestDnsZones` can replicate forest-wide. Administrators can choose other scopes. AD integration does not guarantee healthy DNS; permissions, scavenging, delegation, replication, and client configuration still require management. Secure dynamic updates should normally be used for internal AD zones so only authenticated principals can update authorized records.

## 65. What is the `_msdcs` zone?

**Level:** Intermediate

**Answer:** `_msdcs` contains forest- and domain-controller-locator records, including records based on DC GUIDs and records for services such as Global Catalog and PDC location. In modern forests it is commonly a separate forest-wide AD-integrated zone delegated from the forest-root namespace. Replication and DC discovery can fail when the zone is missing, not delegated correctly, or contains stale records. Validate its replication scope, name servers, delegation, secure updates, and records before recreating data manually.

## 66. What are DNS forwarders, conditional forwarders, stub zones, and delegations?

**Level:** Intermediate

**Answer:** A forwarder sends unresolved queries to another DNS server. A conditional forwarder does so only for a specified namespace and is common in partner or multi-forest environments. A stub zone contains records needed to discover authoritative servers for another zone and updates those references from the target. A delegation tells resolvers that a child namespace is hosted by different authoritative servers. The correct choice depends on ownership, routing, security, and change frequency. In a trust scenario, successful DNS resolution is necessary but does not create the trust or authorization. Avoid bidirectional forwarding loops and undocumented dependencies on a single DNS server.

## 67. What is dynamic DNS registration in a domain?

**Level:** Intermediate

**Answer:** Domain members and DHCP services can dynamically register and refresh host records. Domain controllers also register service-location records through Netlogon. With secure dynamic updates, AD permissions determine who may modify records. Problems arise when DHCP ownership, stale ACLs, duplicate names, multihomed systems, or scavenging configuration conflict. A secure design defines whether clients or DHCP register A and PTR records, uses dedicated DHCP credentials where appropriate, protects sensitive records, and monitors unexpected updates. Restarting a client or running `ipconfig /registerdns` is a diagnostic step, not a substitute for correcting permissions or zone configuration.

## 68. What are DNS aging and scavenging?

**Level:** Intermediate

**Answer:** Aging timestamps dynamically registered records, and scavenging removes stale records after configured no-refresh and refresh intervals plus scavenging timing. It reduces stale DC, client, and application records but can cause outages if enabled without understanding ownership and refresh behavior. Static records are normally exempt unless timestamped. Before enabling scavenging, review DHCP lease periods, zone replication, server scavenging settings, application records, and existing timestamps. Start with reporting and controlled zones. A stale record causing authentication to the wrong server is common; indiscriminate record deletion is not the correct first response.

## 69. Why should domain clients not use public DNS servers as primary resolvers?

**Level:** Beginner

**Answer:** Public DNS resolvers do not host or normally know the private AD service-location records required for the internal domain. A client using a public resolver may resolve internet names while failing to locate domain controllers, leading to misleading partial connectivity. Configure clients to use internal DNS that hosts or can resolve the AD namespace, then configure those internal servers to forward external queries. Adding a public resolver as a “backup” client DNS server is also unsafe because Windows may use it independently rather than only after proving the internal server is permanently unavailable.

## 70. How does a client determine its AD site?

**Level:** Intermediate

**Answer:** The client provides its IP information during DC location, and a domain controller maps that address to the most specific subnet object in the configuration partition. The associated site is returned and cached. If no subnet matches, the client may use a DC from a less optimal site and events may report unmapped addresses. `nltest /dsgetsite` shows the current result. Correct site mapping requires current IPAM data, nonoverlapping subnet definitions, and operational processes that add new networks before deployment. A site name cannot compensate for an incorrect or absent subnet object.

