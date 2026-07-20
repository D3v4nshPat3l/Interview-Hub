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

## 71. Why is time synchronization important in AD?

**Level:** Beginner

**Answer:** Kerberos uses timestamps to reduce replay risk, so clients, services, and KDCs must operate within the configured clock-skew tolerance, commonly five minutes by default. Large drift can cause Kerberos errors, GPO and replication failures, certificate validation issues, and confusing application behavior. Domain members normally synchronize through the domain hierarchy. The PDC Emulator in the forest root should synchronize with a reliable external source, and other DCs and members follow the hierarchy. Virtualization time integration must not fight domain time. Use `w32tm /query /status`, `w32tm /query /source`, and event logs to diagnose the chain.

## 72. What is the recommended Windows time hierarchy in a forest?

**Level:** Intermediate

**Answer:** The forest-root PDC Emulator is the authoritative domain time source and should synchronize with a reliable external source. PDC Emulators in child domains normally synchronize through the parent-domain hierarchy, other domain controllers synchronize with an appropriate domain source, and member computers synchronize from domain controllers. Configuration should be explicit, monitored, and resilient. Never point every domain member directly to an internet NTP service because that breaks the domain hierarchy and complicates Kerberos troubleshooting. After transferring the PDC role, verify the new holder's configuration and source.

## 73. What is a computer-account secure channel?

**Level:** Intermediate

**Answer:** A domain-joined computer has an AD computer account and a corresponding machine password. The workstation or server and the domain use that secret to establish a secure channel for Netlogon operations. The password rotates automatically. If the local and AD copies no longer match, users may see “the trust relationship between this workstation and the primary domain failed.” Causes include stale snapshots, restored images, duplicate computer identities, long-offline systems, account resets, and replication inconsistencies. Use `Test-ComputerSecureChannel`, `nltest /sc_verify`, or `Reset-ComputerMachinePassword` in a controlled manner. Rejoining the domain is not always necessary.

## 74. What happens during a domain join?

**Level:** Intermediate

**Answer:** The device locates a domain controller through DNS, authenticates an authorized joining identity or uses a prestaged account, creates or reuses a computer object, establishes a machine-account password, and configures local domain membership. The process requires correct DNS, time, network ports, naming, computer-account permissions, and a unique identity. Joining does not automatically place the object in the correct OU unless a prestaged object, redirection, or provisioning workflow controls placement. After join, restart and policy processing establish the managed state. Logs such as `C:\Windows\Debug\NetSetup.log` are central to troubleshooting.

## 75. What is an offline domain join?

**Level:** Advanced

**Answer:** Offline domain join provisions domain-join metadata in advance so a computer can become domain-joined without contacting a domain controller during the initial join action. `djoin.exe` can create a provisioning blob that is applied to the target image. This is useful for imaging, remote deployment, and isolated build processes. The blob is sensitive because it contains domain-join material and must be transferred securely, scoped correctly, and deleted after use. The machine still needs domain connectivity later for normal authentication, policy, and password maintenance.

## 76. Which firewall ports are required between clients and domain controllers?

**Level:** Intermediate

**Answer:** Requirements depend on operations, but typically include DNS 53, Kerberos 88, LDAP 389, SMB 445, RPC Endpoint Mapper 135 plus the configured dynamic RPC range, Kerberos password change 464, and NTP 123. GC access uses 3268 or 3269, and LDAPS uses 636 when applications require it. Some workflows also need ICMP and application-specific ports. Between domain controllers, replication, DFSR for SYSVOL, DNS, RPC, and authentication dependencies must be allowed bidirectionally as designed. Use Microsoft's current port guidance and packet captures rather than an incomplete memorized list.

## 77. What is LDAP, and how does AD use it?

**Level:** Beginner

**Answer:** LDAP is the protocol used to search, read, add, modify, and delete directory objects according to permissions. AD DS implements LDAP with Microsoft extensions and exposes directory partitions through LDAP endpoints. Applications can bind using Kerberos, NTLM, certificates, or simple credentials depending on configuration. LDAP does not itself equal authentication; it is a directory-access protocol that can carry different bind mechanisms. Secure designs prefer signed SASL binds or TLS-protected connections, validate certificates for LDAPS, restrict anonymous access, and monitor legacy simple binds.

## 78. What is LDAP signing?

**Level:** Advanced

**Answer:** LDAP signing provides integrity protection for supported LDAP sessions so traffic cannot be modified undetected in transit. Domain controllers can be configured to require signing for SASL binds, and clients can be configured to request or require it. Unsigned binds create relay and man-in-the-middle risk. Enabling enforcement without inventory can break printers, appliances, Java applications, or scripts using simple or unsigned binds. A safe rollout audits relevant domain-controller events, identifies clients, remediates or isolates them, validates channel binding and TLS where applicable, then enforces policy in stages.

## 79. What is LDAP channel binding?

**Level:** Advanced

**Answer:** LDAP channel binding ties authentication at the application layer to the underlying TLS channel by incorporating channel-binding token information. It helps prevent an attacker from relaying credentials from one TLS session to another. It is especially relevant to LDAPS and NTLM relay defenses. Channel binding is distinct from LDAP signing, and the two controls should be assessed together. Compatibility varies by client and application, so use audit modes and current Microsoft guidance before enforcement. Certificate trust, hostname matching, and TLS configuration must also be correct.

## 80. What diagnostic order should you follow for a suspected AD connectivity problem?

**Level:** Intermediate

**Answer:** Start at the affected client and define the failed operation precisely. Check IP configuration, routing, VPN state, DNS server addresses, and time. Resolve the domain and relevant SRV records. Use `nltest /dsgetdc`, `nltest /dsgetsite`, and a TCP connectivity test for required services. Validate the selected DC's health with `dcdiag` and replication with `repadmin`. Check the client, System, Netlogon, DNS, Kerberos, and Directory Service logs as appropriate. Compare a working and failing client. Do not begin by deleting accounts, forcing all replication, or opening every firewall port; preserve evidence and isolate the broken dependency.

---

# Kerberos, NTLM, SPNs, and Delegation

*Windows authentication mechanisms, fallback behavior, tickets, service identities, and constrained impersonation.*

## 81. How does Kerberos authentication work at a high level?

**Level:** Intermediate

**Answer:** The client first obtains a Ticket Granting Ticket from the KDC's Authentication Service, normally after proving knowledge of the user's key during preauthentication. When the client needs a service, it presents the TGT to the Ticket Granting Service and requests a service ticket for the target SPN. The client presents that service ticket to the server, which decrypts its protected portion using the service account's key. The ticket contains authorization data, including a Privilege Attribute Certificate in Windows environments. Kerberos supports mutual authentication and reusable tickets, reducing repeated password validation. DNS, SPNs, time, account keys, and trust relationships must all be correct.

## 82. What is a TGT?

**Level:** Beginner

**Answer:** A Ticket Granting Ticket is a Kerberos credential issued by the KDC after initial authentication. Its encrypted portion is protected with the domain `krbtgt` account's key and allows the client to request service tickets without resending the password. A TGT has a validity period, renewal rules, flags, and authorization information. Possession of a valid TGT is highly sensitive because it represents the user's authenticated session. Password resets may not instantly remove already issued tickets from all systems, so incident response often includes session and ticket invalidation considerations.

## 83. What is a Kerberos service ticket?

**Level:** Beginner

**Answer:** A service ticket is issued for a specific SPN and is presented to the target service. Its server-readable portion is encrypted with the key associated with the service account. That design is why the SPN must be registered on the correct account and why duplicate SPNs produce failures. A service ticket normally contains the client's identity and authorization data but does not give the service the user's password. `klist` can show cached tickets and help determine whether Kerberos was used.

## 84. What is the Kerberos PAC?

**Level:** Advanced

**Answer:** The Privilege Attribute Certificate is Microsoft authorization data carried in Kerberos tickets. It includes information such as the user's SID, group memberships, privileges, and related logon data used to build an access token. The KDC signs the PAC, and services or domain controllers may validate it according to protocol behavior. Forged or manipulated authorization data is central to Golden Ticket and related attacks. Defenders should understand that a Kerberos ticket is not only proof of authentication; it can also convey authorization context that affects access across many systems.

## 85. What is Kerberos preauthentication?

**Level:** Intermediate

**Answer:** Kerberos preauthentication requires the client to prove knowledge of its key before the KDC issues a TGT, commonly by encrypting timestamp data. It prevents unauthenticated retrieval of material that can be tested offline for many accounts. Accounts configured with “Do not require Kerberos preauthentication” are vulnerable to AS-REP roasting because an attacker can request an AS response without first proving knowledge of the password. Exceptions should be rare, documented, monitored, and remediated. Event 4768 and related KDC events help investigate TGT requests.

## 86. What is the difference between Kerberos and NTLM?

**Level:** Beginner

**Answer:** Kerberos is the preferred domain authentication protocol. It uses a KDC and tickets, supports mutual authentication, works with SPNs, and can support delegation. NTLM uses a challenge-response process and does not provide the same server-identity assurance. NTLM commonly appears when Kerberos prerequisites are not met, such as use of an IP address, missing or duplicate SPNs, workgroup access, legacy applications, or certain local-account scenarios. “Kerberos is secure and NTLM is insecure” is too simplistic; both require hardening, but NTLM has greater relay, downgrade, and credential-abuse exposure and should be audited and reduced.

## 87. What is NTLM challenge-response authentication?

**Level:** Intermediate

**Answer:** In NTLM, the server sends a challenge and the client produces a response derived from the account secret and session data. The server or a domain controller validates the response without the plaintext password traversing the network. For domain accounts, the target may use Netlogon to ask a DC to validate the exchange. Captured Net-NTLM responses are not the same as the NT hash stored in AD, but they may be relayed or attacked offline under certain conditions. Protections include reducing NTLM, requiring signing or channel binding, Extended Protection, SMB signing, strong passwords, and eliminating coercion paths.

## 88. Under what conditions does Windows fall back to NTLM?

**Level:** Intermediate

**Answer:** Fallback can occur when the client cannot identify a valid SPN, connects by IP address, accesses a system outside a Kerberos trust path, uses a local account, encounters a legacy protocol or application, or fails Kerberos negotiation. The Negotiate security package normally attempts Kerberos and then NTLM when appropriate. Fallback should be treated as a diagnostic signal rather than accepted blindly. Use service logs, `klist`, packet capture, SPN checks, and NTLM auditing to determine why Kerberos was not selected. Disabling NTLM before correcting dependencies can produce widespread outages.

## 89. What is `KRB_AP_ERR_MODIFIED`?

**Level:** Advanced

**Answer:** This Kerberos error usually means the service receiving a ticket cannot decrypt it with the key it possesses. Common causes are a duplicate SPN on another account, an SPN registered on the wrong account, a service-account password mismatch, a stale machine account, load-balanced servers using inconsistent identities, or traffic reaching an unexpected host. Identify the requested SPN, the account that owns it, the account actually running the service, and whether all service instances share the intended key. Do not reset random passwords before collecting this information because that can hide the root cause.

## 90. How do you troubleshoot duplicate SPNs?

**Level:** Intermediate

**Answer:** Determine the exact SPN requested by the client and search the forest with `setspn -Q <SPN>` or use `setspn -X` for duplicate detection. Confirm which identity runs the service, whether a computer account already has a host-based SPN, and whether DNS aliases or old service accounts introduced duplicates. Remove only the incorrect registration and add the correct one with `setspn -S`. Then purge or renew tickets and test from a clean session. In a cluster or farm, use a shared managed service account or properly designed service identity rather than registering the same SPN on unrelated accounts.

## 91. What is Kerberoasting?

**Level:** Advanced

**Answer:** Kerberoasting abuses the legitimate ability of an authenticated principal to request service tickets for SPNs. Part of the ticket is protected with the service account's key, so an attacker can attempt offline password guessing. Risk is highest for user-based service accounts with weak, old, human-managed passwords and RC4 compatibility. Defenses include gMSA or dMSA adoption, long random passwords, AES support, least privilege, removal of unnecessary SPNs, separate service identities, and monitoring unusual volumes or patterns of service-ticket requests such as event 4769. A successful roast does not itself grant more privilege than the cracked service account possesses, so account rights determine impact.

## 92. What is AS-REP roasting?

**Level:** Advanced

**Answer:** AS-REP roasting targets accounts that do not require Kerberos preauthentication. An attacker can request an authentication response for such an account without valid credentials and then attempt offline password guessing against encrypted response material. Detect accounts with the relevant `userAccountControl` flag, remove the exception unless a documented legacy requirement exists, set strong random passwords, and monitor unusual TGT requests. The main preventative control is to require preauthentication.

## 93. What is Pass-the-Hash?

**Level:** Advanced

**Answer:** Pass-the-Hash is the use of an NT password hash as authentication material without first recovering the plaintext password. It is associated mainly with NTLM-based authentication and administrative protocols. The attack demonstrates why password hashes must be protected as credential equivalents. Defenses include Windows LAPS for unique local administrator passwords, Credential Guard where compatible, restricting admin logon, reducing NTLM, disabling or limiting remote administrative protocols, using secure administrative hosts, segmenting Tier 0, and monitoring anomalous authentication. A password reset invalidates future use of the old hash but may not remove established sessions or other credentials.

## 94. What is Pass-the-Ticket?

**Level:** Advanced

**Answer:** Pass-the-Ticket uses a stolen Kerberos TGT or service ticket in another logon context. The attacker does not need the plaintext password while the ticket is valid. Ticket theft can occur from compromised endpoints, memory, caches, or improperly protected files on Windows or Linux. Mitigations include preventing credential exposure on low-trust hosts, using privileged-access tiers, Credential Guard, short and appropriate ticket lifetimes for sensitive identities, Protected Users where compatible, endpoint detection, and rapid containment. Investigators correlate ticket use, source devices, logon sessions, KDC events, and endpoint telemetry.

## 95. What is a Golden Ticket?

**Level:** Advanced

**Answer:** A Golden Ticket is a forged TGT created using key material for the domain's `krbtgt` account. It can contain manipulated identity and authorization data and may provide durable domain-level persistence. Its existence normally implies severe compromise of domain secrets or a domain controller. Response requires more than resetting one user: contain privileged access, investigate the compromise path, reset `krbtgt` twice with adequate replication and planning, rotate other exposed credentials, review trusts and certificates, and consider forest recovery when integrity cannot be established. Detection combines ticket anomalies, encryption behavior, impossible identity data, endpoint evidence, and identity-threat analytics.

## 96. What is a Silver Ticket?

**Level:** Advanced

**Answer:** A Silver Ticket is a forged service ticket created with the key of a particular service account or computer account. It normally targets services associated with that identity and may avoid contacting the KDC during use, reducing KDC-side visibility. Impact depends on the service and account privileges. Response includes rotating the compromised service or machine account key, correcting service identity design, reviewing the host, and investigating how the key was obtained. Managed service accounts and machine-bound approaches reduce risks associated with static human-managed service credentials.

## 97. What is unconstrained delegation?

**Level:** Advanced

**Answer:** Unconstrained delegation allows a service to receive a forwardable representation of a user's credentials so it can access other services on the user's behalf. On a compromised delegated host, privileged users' tickets may be exposed, making this configuration high risk. Inventory accounts and computers trusted for unconstrained delegation, remove it where possible, avoid privileged sign-in to those systems, use Protected Users or “account is sensitive and cannot be delegated” for compatible privileged identities, and migrate applications to constrained or resource-based models. Domain controllers have special delegation-related behavior and require separate consideration.

## 98. What is constrained delegation?

**Level:** Advanced

**Answer:** Constrained delegation limits the services to which an account may delegate, represented through attributes such as `msDS-AllowedToDelegateTo`. Protocol transition can allow a service to obtain a Kerberos identity for a user who did not initially authenticate with Kerberos, depending on configuration. It is safer than unconstrained delegation but still powerful: compromise of the delegating account can permit impersonation to listed services. Review target SPNs, protocol-transition settings, service-account privilege, and whether the application actually requires delegation. Prefer least-privilege service identities and monitor changes.

## 99. What is resource-based constrained delegation?

**Level:** Advanced

**Answer:** Resource-based constrained delegation, or RBCD, places the authorization decision on the target resource through `msDS-AllowedToActOnBehalfOfOtherIdentity`. This simplifies cross-domain and service-owner administration because the resource owner controls which principals may act on behalf of users. Security depends on who can modify the target computer or service object's attribute. An apparently low-level right such as control over a computer object can therefore become an impersonation path. Audit write permissions, machine-account creation, RBCD attribute changes, and service tickets. Remove stale entries after migrations or testing.

## 100. What are Protected Users and “Account is sensitive and cannot be delegated”?

**Level:** Advanced

**Answer:** Protected Users is a group that applies stronger credential protections to compatible domain accounts, including restrictions on NTLM, weaker Kerberos encryption, credential caching, and delegation. It can break legacy applications and should be tested before broad use. The “Account is sensitive and cannot be delegated” flag prevents supported delegation of that account but does not provide every protection of Protected Users. Both are useful for privileged identities, but neither replaces tiering, secure administration, MFA at appropriate access layers, endpoint hardening, and credential rotation. Emergency and service accounts require compatibility analysis before enrollment.

---
# Group Policy and SYSVOL

*Policy architecture, processing, replication, security filtering, and troubleshooting.*

## 101. What is a Group Policy Object?

**Level:** Beginner

**Answer:** A Group Policy Object, or GPO, is a collection of computer and user configuration settings that can be linked to sites, domains, and OUs. Each domain GPO has two coordinated components: the Group Policy Container in AD, which stores metadata and version information, and the Group Policy Template in SYSVOL, which stores files such as registry policy data, scripts, and administrative template settings. Because these components replicate through different mechanisms, a GPO can become inconsistent if AD replication or SYSVOL DFSR replication fails. GPOs are applied based on scope, link order, inheritance, security filtering, WMI filters, and client-side extension behavior.

## 102. What is the difference between a GPO link and a GPO?

**Level:** Beginner

**Answer:** A GPO is the policy object itself; a link associates that GPO with a site, domain, or OU. One GPO can have multiple links, and deleting a link does not necessarily delete the GPO. Conversely, deleting the GPO affects every link that references it. Each link has properties such as enabled state, enforced state, and link order. Troubleshooting must identify both the GPO's content and the particular link path through which it should apply. A common mistake is editing a GPO because its name appears relevant without checking whether the affected user or computer is actually within a linked scope.

## 103. Explain LSDOU Group Policy processing order.

**Level:** Beginner

**Answer:** The standard processing order is Local, Site, Domain, and then OU, with nested OUs processed from parent to child. Later applicable settings generally win when the same policy setting is configured more than once, subject to exceptions such as enforcement, blocked inheritance, security filtering, loopback processing, and client-side extension rules. Link order also matters within the same container. LSDOU is a starting model, not the complete evaluation algorithm. A correct troubleshooting answer uses Resultant Set of Policy or `gpresult` rather than relying only on visual inspection of GPMC.

## 104. What are Enforced and Block Inheritance?

**Level:** Intermediate

**Answer:** Block Inheritance on a domain or OU prevents ordinary GPO links from parent containers from applying below that point. Enforced, historically called No Override, causes a GPO link to continue applying through blocked inheritance and prevents lower-level GPOs from overriding its configured settings. The Local GPO is not controlled through domain link enforcement. Overusing Enforced and Block Inheritance produces policy designs that are difficult to troubleshoot and delegate. Security baselines should be intentionally layered, documented, and tested rather than relying on widespread enforcement to compensate for poor OU design.

## 105. What is security filtering in Group Policy?

**Level:** Intermediate

**Answer:** Security filtering uses the GPO's ACL to determine which users or computers have both Read and Apply Group Policy permissions. By default, Authenticated Users commonly provides broad read and apply scope. Removing it without preserving appropriate read permissions can create unexpected failures, especially for computer-side processing of user policies. Use dedicated security groups, understand whether the target is a user or computer, and validate the effective ACL. Security filtering does not change the linked container; it narrows which objects within the scope apply the GPO.

## 106. What is a WMI filter?

**Level:** Intermediate

**Answer:** A WMI filter is a query evaluated by the client to determine whether a linked GPO should apply. It can target operating-system version, hardware characteristics, or other WMI data. WMI filters add processing cost and can fail because of provider issues, query errors, or slow evaluation. Prefer item-level targeting or clear OU and group design when those methods are sufficient. When troubleshooting, confirm the filter result in `gpresult`, test the query locally, and consider namespaces, permissions, and operating-system changes.

## 107. What is Group Policy loopback processing?

**Level:** Advanced

**Answer:** Loopback processing changes how user policy is calculated based on the computer being used. In Merge mode, the normal user GPOs are processed and then the user settings from the computer's GPO path are added, with the latter generally taking precedence. In Replace mode, the user's normal OU-based GPO list is replaced by the user settings associated with the computer's path. Loopback is common for kiosks, RDS hosts, classrooms, and shared administrative systems. It should be scoped to the relevant computers, documented, and validated because it can make user-policy behavior appear inconsistent across devices.

## 108. What is the Default Domain Policy intended for?

**Level:** Intermediate

**Answer:** The Default Domain Policy is linked at the domain and should generally be kept focused on domain-wide account policy and a small set of truly domain-wide settings. It is not a mandatory location for every security configuration. Adding unrelated workstation, server, browser, and application settings makes recovery and troubleshooting harder. Create separate, purpose-built GPOs with clear ownership and scope. Do not unlink or casually replace the default policy; if it is damaged, supported tools such as `dcgpofix` may restore defaults, but that does not restore custom settings and should not be used as a first-line fix.

## 109. What is the Default Domain Controllers Policy intended for?

**Level:** Intermediate

**Answer:** The Default Domain Controllers Policy is linked to the Domain Controllers OU and provides baseline settings relevant to domain controllers, including certain user rights and auditing defaults. It does not store passwords or define every domain-controller security control. Organizations should use separate tested GPOs for additional DC hardening while preserving required rights and understanding precedence. Removing critical rights from built-in groups can break services such as DFSR SYSVOL initialization. Changes to domain-controller policy require lab validation, staged deployment, and rollback planning.

## 110. What is SYSVOL?

**Level:** Beginner

**Answer:** SYSVOL is a replicated folder present on each domain controller in a domain. It contains the file-based portion of GPOs, sign-in and startup scripts, and other public domain files required by clients. The `SYSVOL` and `NETLOGON` shares expose relevant content. Current domains use DFS Replication for SYSVOL. The AD database and transaction logs are not stored in SYSVOL. Healthy Group Policy requires both directory replication of the GPC and DFSR replication of the GPT.

## 111. What is the NETLOGON share?

**Level:** Beginner

**Answer:** NETLOGON is a share exposed by domain controllers that maps to the scripts portion of SYSVOL and is traditionally used for sign-in scripts and related files. Its absence on a new or existing DC often indicates incomplete SYSVOL initialization, DFSR failure, or Netlogon not advertising the DC as ready. Do not manually create the share as a permanent workaround. Investigate DFS Replication events, AD replication, the DC's SYSVOL subscription, upstream health, and promotion status.

## 112. How is SYSVOL replicated?

**Level:** Intermediate

**Answer:** Supported current domains use Distributed File System Replication, or DFSR, to replicate SYSVOL. Older environments may still contain legacy FRS configurations, which should be migrated before introducing unsupported newer domain-controller versions. DFSR has its own service state, event logs, database, staging behavior, conflict handling, and AD configuration. AD replication must also be healthy because DFSR configuration is stored in AD. Commands such as `dfsrmig /getglobalstate`, `dfsrmig /getmigrationstate`, `dfsrdiag pollad`, and event-log review help establish state.

## 113. Why can a GPO show different versions in AD and SYSVOL?

**Level:** Advanced

**Answer:** The directory component and file component replicate independently. An edit increments version data, but AD replication may converge while DFSR is delayed or broken, or vice versa. A client can therefore read metadata from one DC and files from another state, causing processing errors or inconsistent settings. Compare GPO version information, identify the DC used by the client, check `repadmin`, inspect DFS Replication logs and backlog, and verify the GPT path and `gpt.ini`. Avoid manually copying individual policy files between DCs because that can create unsupported divergence.

## 114. What is `gpupdate`?

**Level:** Beginner

**Answer:** `gpupdate` triggers a policy refresh on the local computer. `/force` reapplies all settings rather than only changed GPOs, and `/boot` or `/logoff` can be used when a client-side extension requires restart or sign-out. `gpupdate` does not repair replication, correct scoping, or make an inaccessible DC healthy. Repeatedly forcing policy can obscure timing and increase load. First determine whether the GPO reached the DC being used and whether the client considers it applicable.

## 115. What is `gpresult`, and how is it used?

**Level:** Beginner

**Answer:** `gpresult` reports Resultant Set of Policy information for a user or computer. `gpresult /r` provides a summary, while `gpresult /h report.html` creates a detailed report showing applied and denied GPOs, filtering reasons, winning settings, group membership, and processing data. Run it in the affected security context and with elevation for computer details. A report from an administrator's session may not represent the user's session. Pair it with the GroupPolicy Operational event log and GPMC modeling when diagnosing remote or future-state behavior.

## 116. What is RSoP?

**Level:** Intermediate

**Answer:** Resultant Set of Policy is the calculated outcome of all applicable policy inputs. Logging mode reports what applied to a specific user and computer. Planning mode models what would apply under hypothetical OU, group, site, or loopback conditions, subject to data availability. RSoP helps explain precedence but does not prove that every client-side extension executed successfully or that a script completed. Operational logs and application-specific evidence are still required.

## 117. Why might a GPO be listed as denied?

**Level:** Intermediate

**Answer:** Common reasons include security filtering, a false WMI filter, disabled user or computer configuration, an unlinked or disabled link, object location outside scope, loopback behavior, inaccessible SYSVOL files, or replication mismatch. `gpresult` normally states the denial reason. Also check whether the target's group token includes a newly added group, because membership changes often require sign-out or restart. Do not solve every denial by granting Authenticated Users Apply Group Policy; that can broaden scope beyond the design.

## 118. What are Group Policy Preferences?

**Level:** Intermediate

**Answer:** Group Policy Preferences configure settings such as drive mappings, printers, scheduled tasks, files, folders, services, and registry values. Unlike enforced policy settings, many preferences tattoo the system and may remain after the GPO no longer applies unless removal behavior is configured. Item-level targeting provides granular conditions. Preferences are powerful and should be treated as code: peer review changes, minimize credential exposure, validate action modes such as Create, Replace, Update, and Delete, and monitor privileged task or group changes. Legacy Group Policy Preference password storage using `cpassword` is insecure and should be removed.

## 119. What was the Group Policy Preferences `cpassword` issue?

**Level:** Advanced

**Answer:** Older Group Policy Preferences could store encrypted passwords in XML files in SYSVOL using a publicly disclosed static key. Any authenticated domain user who could read SYSVOL could recover those passwords. Microsoft removed the UI ability to create new password-bearing preferences, but legacy XML may remain in historical or active policy paths. Search SYSVOL for `cpassword`, identify affected accounts, remove the configuration, rotate credentials, and investigate where the account was used. This is a classic example of why readable configuration data should never contain reusable privileged secrets.

## 120. What is a safe GPO change process?

**Level:** Advanced

**Answer:** Define the objective and affected population, clone or back up the GPO, review current links and ACLs, test in a representative lab or pilot OU, use change control and peer review, and establish success and rollback criteria. Separate user and computer settings when useful, disable unused halves, avoid editing broad production GPOs for one exception, and record owners and purpose. After deployment, confirm AD and SYSVOL replication, collect `gpresult` from pilot systems, monitor events and business outcomes, then expand gradually. Security-sensitive GPOs should be protected from unauthorized modification and monitored for change.

---

# Replication, Sites, Trusts, Backup, and Recovery

*Directory convergence, topology, conflict handling, forest relationships, and resilient operations.*

## 121. How does AD replication work?

**Level:** Intermediate

**Answer:** AD replication transfers changes to directory partitions between domain controllers that host those partitions. Each writable DC can originate most updates. Replication metadata includes originating DC identity, invocation ID, version, update sequence numbers, and timestamps. Destination DCs track high-watermark and up-to-dateness vectors to avoid resending or reapplying changes. Intrasite replication is optimized for speed; intersite replication follows site links, costs, schedules, and bridgehead decisions. Replication depends on DNS, RPC connectivity, authentication, time, and a topology generated by the KCC.

## 122. What is the KCC?

**Level:** Intermediate

**Answer:** The Knowledge Consistency Checker runs on domain controllers and automatically generates replication connection objects to create a resilient topology. Within a site, it builds efficient connections with redundancy. Between sites, the Inter-Site Topology Generator selects connections based on site links, costs, schedules, and partition availability. Administrators normally manage sites, subnets, and site links rather than manually constructing every connection. Persistent manual connections can interfere with adaptation and must be documented. KCC errors often point to missing connectivity, DNS, site-link, or partition information.

## 123. What is the ISTG?

**Level:** Advanced

**Answer:** The Inter-Site Topology Generator is the KCC role performed by one DC in each site to build intersite replication connections. It determines bridgehead relationships for partitions based on site topology. The role can move automatically if the current ISTG is unavailable. Troubleshooting an intersite topology requires identifying the ISTG, site-link configuration, bridgehead restrictions, schedules, costs, and whether the required partition has a replica in the sites involved. Manually selecting preferred bridgeheads can reduce resilience if too few servers are available.

## 124. What is the difference between intrasite and intersite replication?

**Level:** Beginner

**Answer:** Intrasite replication assumes fast, reliable connectivity and is change-notification driven, compressed only where protocol behavior requires, and optimized for quick convergence. Intersite replication is designed for WAN links and follows site-link schedules and intervals, uses cost to select paths, and compresses data. A single physical building can contain multiple sites if network behavior justifies it, and multiple buildings can be one site if well connected. Design from measured network and service requirements, not office names.

## 125. What is a site link?

**Level:** Beginner

**Answer:** A site link represents available network connectivity between two or more AD sites for replication. It has a transport, cost, schedule, and replication interval. Lower cost is preferred when multiple routes exist. A site link is not a router configuration and does not create network reachability; the underlying network must already support the traffic. Model actual connectivity, avoid placing every site in the default site link, and ensure schedules overlap if transitive paths are expected.

## 126. What is site-link bridging?

**Level:** Advanced

**Answer:** Site-link bridging treats site links as transitive so the KCC can construct replication paths through combined links. “Bridge all site links” is enabled by default for a fully routed IP network. If the network is not fully routed, leaving automatic bridging enabled can cause the KCC to create impossible paths. In that case, disable automatic bridging and create explicit site-link bridges that represent real routing. The design must account for schedules, costs, firewalls, and whether an intermediate site hosts a replica of the partition.

## 127. What is a bridgehead server?

**Level:** Intermediate

**Answer:** A bridgehead server is a domain controller selected to send or receive intersite replication for a partition and transport. The KCC normally selects bridgeheads dynamically. Administrators can configure preferred bridgeheads, but doing so restricts automatic choices and may create a single point of failure if the selected systems are unavailable or do not host needed partitions. Prefer automatic selection unless a validated network, firewall, or capacity requirement justifies control.

## 128. What does `repadmin /replsummary` show?

**Level:** Beginner

**Answer:** `repadmin /replsummary` summarizes inbound and outbound replication status, including failure counts, largest replication deltas, and error percentages. It is a rapid forest or domain health view, not a complete diagnosis. Follow failures with `repadmin /showrepl`, `/showconn`, `/showobjmeta`, DNS checks, event logs, and network tests. A zero-failure summary at one moment does not prove SYSVOL, DNS application partitions, or application behavior is healthy.

## 129. What does `repadmin /showrepl` show?

**Level:** Intermediate

**Answer:** `repadmin /showrepl` reports inbound replication partners for each naming context, the last attempt, last success, consecutive failures, and status codes. It helps identify whether failure is limited to one partition, one source, one destination, or a broad dependency. Use `/csv` for larger analysis and compare timestamps to expected schedules. Interpret errors in context: access denied, RPC unavailable, target principal name incorrect, lingering objects, and DNS failures require different remediation.

## 130. What is `dcdiag`?

**Level:** Beginner

**Answer:** `dcdiag` runs diagnostic tests against domain controllers, including advertising, services, replication, DNS, connectivity, system logs, roles, and other health areas depending on options. `dcdiag /v`, `dcdiag /e`, and `dcdiag /test:dns` are common. Some warnings are environmental or historical, so do not treat every line as a critical failure without understanding the test. Capture output before and after remediation and correlate with `repadmin`, events, DNS, and actual symptoms.

## 131. What are USNs and invocation IDs?

**Level:** Advanced

**Answer:** A domain controller assigns an update sequence number to each local directory change. USNs are meaningful only relative to that DC's current database instance, identified by its invocation ID. Replication partners track how far they have received changes from each source. After a supported restore or generation change, AD changes the invocation ID so old and new update sequences are not confused. This mechanism is central to detecting unsafe rollback and ensuring convergence. Never compare raw USNs across different DCs as if they were global transaction numbers.

## 132. What is USN rollback?

**Level:** Advanced

**Answer:** USN rollback occurs when a DC database is reverted without AD being able to recognize the restore correctly. The DC may reuse update sequence numbers while partners believe those changes were already received, causing silent divergence. Modern VM-GenerationID and AD-aware backup processes reduce risk, but unsupported snapshots, storage rollback, or copied images remain dangerous. Symptoms can include replication being disabled and directory-service events. The usual response is to isolate and rebuild or perform supported recovery, not force replication from the rolled-back DC.

## 133. What are lingering objects?

**Level:** Advanced

**Answer:** Lingering objects are objects that remain on a DC after they were deleted elsewhere and the deletion tombstone expired before the stale DC replicated it. When the stale DC returns, it may attempt to reintroduce or serve those objects. Strict replication consistency helps block inbound replication of such data. Diagnose long replication gaps, compare authoritative replicas, use `repadmin /removelingeringobjects` only with correct source and advisory checks, and rebuild a DC when confidence is low. Extending tombstone lifetime after the problem does not retroactively repair it.

## 134. What is tombstone lifetime?

**Level:** Advanced

**Answer:** Tombstone lifetime controls how long deletion information is retained for replication and influences the maximum safe offline or backup age. Defaults vary with forest creation history and configuration; many newer forests use 180 days, while protocol defaults and older forests may show 60 days. Always query the actual `tombstoneLifetime` attribute and understand `msDS-DeletedObjectLifetime` when Recycle Bin is enabled. Backups older than the effective safe lifetime should not be assumed valid for ordinary restore. Monitoring should alert well before any DC approaches the limit without successful inbound replication.

## 135. What is Active Directory Recycle Bin?

**Level:** Intermediate

**Answer:** Recycle Bin is a forest-wide optional feature that preserves deleted objects with their link-valued and non-link-valued attributes during the deleted-object lifetime, allowing restoration with group memberships and object identity intact. It requires an appropriate functional level and cannot be disabled after enablement. It only helps for objects deleted after it was enabled. It is not a substitute for system-state backup or forest recovery because it does not recover every corruption, compromise, GPO file, DNS issue, or deleted data outside AD objects.

## 136. What is authoritative versus non-authoritative restore?

**Level:** Advanced

**Answer:** In a non-authoritative restore, a DC is restored from backup and then receives newer replicated changes from partners, making it current. In an authoritative restore, selected objects or a subtree are marked with higher version metadata so the restored data replicates outward and overrides existing replicas. Modern object deletion recovery should normally use Recycle Bin when available. Authoritative restore is a controlled disaster-recovery procedure requiring DSRM, supported backup, healthy replication design, and careful handling of linked attributes. Forest-wide compromise may require forest recovery rather than object-level authoritative restore.

## 137. What is system-state backup for a domain controller?

**Level:** Intermediate

**Answer:** System state includes the components required to recover the server's directory role, such as AD DS data and logs, registry, boot files, SYSVOL, and other role-dependent state. Use an AD-aware supported backup product and test restoration. Protect backup credentials and repositories as Tier 0 assets because they can contain domain secrets. Backup age must remain within safe directory lifetimes, and at least one recoverable DC per domain should be protected according to the forest recovery plan. A hypervisor snapshot alone is not a backup strategy.

## 138. What is an Active Directory forest recovery plan?

**Level:** Advanced

**Answer:** A forest recovery plan defines how to restore a trusted forest after catastrophic corruption or compromise. It identifies the first writable DC to recover in the forest root, DNS and time dependencies, backup selection, isolation network, role handling, password and key resets, recovery of additional domains, validation gates, and reintroduction of member systems. It includes clean administrative credentials, offline documentation, backup integrity tests, communication ownership, and business priorities. The plan must be rehearsed because forest recovery is procedural and sequencing errors can reintroduce compromised or inconsistent state.

## 139. What is a trust relationship?

**Level:** Beginner

**Answer:** A trust allows authentication from one domain or forest to be accepted for potential access to resources in another. The trusting side owns the resources and accepts identities from the trusted side. Trust direction is opposite the direction of permitted authentication flow: if Domain A trusts Domain B, users from B can potentially access authorized resources in A. A trust does not grant access by itself; resource ACLs and policies still decide authorization. Trusts can be one-way or two-way, transitive or nontransitive, and have types such as parent-child, tree-root, external, forest, shortcut, or realm.

## 140. What is trust transitivity?

**Level:** Intermediate

**Answer:** A transitive trust can extend authentication relationships through connected trusted domains according to trust type and forest rules. Parent-child and tree-root trusts within a forest are two-way and transitive. External trusts are normally nontransitive. Forest trusts are transitive between the forest root namespaces within configured constraints, but they do not merge the forests into one security boundary. Selective authentication, SID filtering, name suffix routing, and resource ACLs still shape access. Never infer effective access from a diagram alone; validate direction, transitivity, and authorization.

## 141. What is a forest trust?

**Level:** Advanced

**Answer:** A forest trust connects two AD forests and can be one-way or two-way. It supports Kerberos authentication across forest boundaries and name-suffix routing for forest namespaces. It is created between forest root domains and can be configured with forest-wide or selective authentication. The forests retain separate schemas, configurations, administrators, and recovery boundaries. DNS resolution, time, network paths, SID filtering, and application requirements must be planned. A two-way trust expands reachable attack paths and should be reviewed like an external security integration.

## 142. What is selective authentication?

**Level:** Advanced

**Answer:** Selective authentication requires administrators in the trusting forest or domain to explicitly grant “Allowed to authenticate” on the target computers or services to principals from the trusted side. Without it, forest-wide authentication permits trusted users to attempt authentication to broadly reachable resources, although ACLs still control access. Selective authentication reduces exposure but increases administrative complexity. Design resource groups, automate grants, monitor failures, and test service accounts and management tools before enforcement.

## 143. What is SID filtering on a trust?

**Level:** Advanced

**Answer:** SID filtering removes or rejects SIDs in cross-trust authorization data that should not be accepted from the other side, reducing SIDHistory-based privilege abuse. Behavior depends on trust type and configuration. Disabling filtering can be required in narrow migration scenarios but creates significant risk and should be temporary, documented, monitored, and reversed. Do not confuse SID filtering with ordinary resource ACL filtering; it validates authorization data crossing the trust.

## 144. What is a shortcut trust?

**Level:** Intermediate

**Answer:** A shortcut trust is an explicitly created transitive trust between domains in the same forest to shorten the authentication path when users frequently access resources across a deep domain hierarchy. It can improve performance in complex legacy forests. It does not create a new security boundary or replace correct DNS and site design. Modern simpler forests rarely require it, and the operational benefit should be measured before adding trust complexity.

## 145. How are password changes replicated?

**Level:** Advanced

**Answer:** Password changes are written to the DC handling the request and receive urgent replication behavior to improve convergence. The PDC Emulator is involved in special password-validation behavior: when another DC rejects a password, it may consult the PDC before final failure, and password changes are sent promptly toward the PDC. This reduces failures immediately after a change but does not eliminate all site or connectivity effects. Troubleshoot which DC processed the change, replication status, PDC availability, and cached credentials rather than assuming global instant consistency.

## 146. What is urgent replication?

**Level:** Advanced

**Answer:** Urgent replication is change notification triggered for security-sensitive updates, such as certain password and account lockout changes, so partners are notified without waiting for ordinary intervals. It still depends on available topology, connectivity, and replication scope. The term does not mean every DC in every site receives the change simultaneously. Intersite design and notification configuration affect propagation. Use metadata and event timing to validate actual convergence.

## 147. What is conflict resolution in AD replication?

**Level:** Advanced

**Answer:** When conflicting changes occur, AD compares attribute or object version metadata, originating timestamps, and originating DC GUIDs to select a winner. Different attributes on the same object can merge because replication is primarily attribute-based. Concurrent object creation with the same name may produce conflict-mangled names containing `CNF` and a GUID. Conflict resolution ensures deterministic convergence but does not guarantee the business-correct value wins. Investigate the origin, automation, permissions, and replication timing before cleaning up conflicts.

## 148. What is linked-value replication?

**Level:** Advanced

**Answer:** Linked-value replication tracks individual values of multivalued linked attributes, such as group membership, rather than replicating the entire membership list as one unit. It improves convergence and reduces conflicts for large groups. Membership metadata can be inspected to determine when and where a member was added or removed. This is valuable in incident response and troubleshooting, but retention and garbage collection limit historical visibility. Record critical group changes in centralized logs as well.

## 149. How do you check replication metadata for an object?

**Level:** Intermediate

**Answer:** Use `repadmin /showobjmeta <DC> <distinguished-name>` or PowerShell such as `Get-ADReplicationAttributeMetadata`. Review attribute version, originating DC, originating time, local USN, and invocation ID. For group membership, use linked-value metadata options or appropriate PowerShell. Metadata helps distinguish which DC originated a change and whether replicas received it. Time fields depend on clock quality, and metadata does not identify the human actor by itself; correlate with directory-service auditing, administrative logs, and endpoint evidence.

## 150. What are application directory partitions?

**Level:** Intermediate

**Answer:** Application partitions allow applications to store directory data with a customized set of domain-controller replicas instead of replicating to every DC in a domain or forest. AD-integrated DNS uses application partitions such as `DomainDnsZones` and `ForestDnsZones`. They can contain dynamic objects but not security principals in the normal domain-account sense. Troubleshooting requires checking which DCs hold a replica, partition cross-references, DNS enlistment, and replication status. A DC can be healthy for the domain partition while missing an application partition required for DNS.

## 151. How should stale domain controllers be handled?

**Level:** Advanced

**Answer:** Determine how long the DC has been offline compared with tombstone lifetime, whether it holds unique roles or services, and whether its database and operating system can be trusted. Do not simply power on a DC that exceeded safe replication age. Isolate it, preserve needed evidence, and normally perform metadata cleanup followed by rebuilding from a supported server version. Remove stale DNS, server, connection, and replication objects carefully. If it held FSMO roles, transfer or seize them according to recovery criteria.

## 152. What is metadata cleanup?

**Level:** Intermediate

**Answer:** Metadata cleanup removes AD references to a permanently failed or forcibly removed domain controller, including its NTDS Settings and server objects and related connection metadata. Modern graphical tools can perform much of the cleanup, and `ntdsutil` remains available. Afterward, verify DNS records, DFSR membership, sites, replication, FSMO roles, and monitoring inventory. Metadata cleanup does not erase the failed server's disks or revoke every credential that may have existed on it; secure disposal and incident response are separate tasks.

## 153. What is `ntdsutil` used for?

**Level:** Intermediate

**Answer:** `ntdsutil` is an advanced AD DS administration and recovery tool. Uses include metadata cleanup, FSMO seizure, authoritative restore workflows, database maintenance, snapshot management, and certain password or partition tasks. It can cause major damage when used without a validated procedure. Prefer task-specific PowerShell or graphical tools for routine work, capture backups and current state, and use Microsoft guidance appropriate to the server version. Commands copied from Windows 2000-era material may be obsolete or incomplete.

## 154. What is offline defragmentation of `ntds.dit`?

**Level:** Advanced

**Answer:** Offline defragmentation creates a compacted copy of the AD database while AD DS is offline, potentially reclaiming disk space. Normal online garbage collection reuses free space internally but usually does not shrink the file. Offline compaction is rarely needed solely for performance and introduces downtime and recovery risk. First measure free pages, disk capacity, backup state, database integrity, and root cause of growth. In many environments, adding storage or rebuilding a DC is safer than manipulating the database.

## 155. What is semantic database analysis?

**Level:** Advanced

**Answer:** Semantic database analysis checks internal directory data consistency beyond file-level ESE integrity. It can identify or repair certain inconsistencies under controlled recovery guidance. It is not a routine maintenance command and should not be used to “optimize” a healthy DC. Before any database repair, preserve backups, diagnose hardware and storage, compare other replicas, and consult current vendor guidance. Because AD is replicated, rebuilding a nonunique DC is often safer than repairing an uncertain database.

## 156. What is the AD database 32K page feature in Windows Server 2025?

**Level:** Expert

**Answer:** Windows Server 2025 introduces support for a 32K ESE page format to improve AD DS and AD LDS scalability, including larger multivalued attributes. New installations are 32K-capable but may operate in an 8K simulation mode for compatibility. Enabling the forest-wide optional feature requires all DCs to be Windows Server 2025 or later, appropriate functional levels, 32K-capable databases, healthy replication, tested backups, and application compatibility validation. In-place upgraded DCs retain their existing 8K database format until rebuilt or otherwise made capable according to supported guidance. This is an architectural migration, not a casual performance toggle.

## 157. What should be monitored daily in AD?

**Level:** Intermediate

**Answer:** Monitor domain-controller availability and advertising, replication failures and largest deltas, DNS service and SRV registration, SYSVOL and NETLOGON shares, DFSR events, time-source health, disk space, backup success, certificate expiry, privileged group and ACL changes, account lockout anomalies, authentication failure trends, stale DCs, and security alerts. Include AD CS, Entra Connect, AD FS, hypervisors, and backup systems if they are part of the identity control plane. Monitoring must have owners, thresholds, escalation paths, and regular test alerts; dashboards without response procedures do not create resilience.

## 158. How often should AD recovery be tested?

**Level:** Advanced

**Answer:** Test frequency depends on risk and change rate, but critical organizations should perform component restores regularly and a full forest recovery exercise at a defined interval, commonly at least annually and after major architectural change. Tests should use isolated infrastructure, known-good backups, documented timings, dependency recovery, credential rotation, and business validation. A successful backup job is not evidence of recoverability. Record lessons, update runbooks, and track whether recovery objectives were met.

## 159. What is the difference between high availability and disaster recovery for AD?

**Level:** Intermediate

**Answer:** High availability uses multiple healthy domain controllers, sites, DNS servers, and resilient infrastructure so service continues through ordinary failures. Disaster recovery restores trusted service after catastrophic loss, corruption, ransomware, or compromise. Replication provides availability but also replicates harmful deletions and malicious changes; it is not backup. A resilient design combines multiple DCs with isolated, immutable, tested backups and a forest recovery process.

## 160. What makes a domain controller a Tier 0 asset?

**Level:** Intermediate

**Answer:** A domain controller can authenticate identities, modify directory data, issue Kerberos tickets, replicate credential material, and influence policy for domain systems. Control of a DC usually enables control of the domain and can threaten the forest. Tier 0 also includes systems and identities that can control DCs indirectly: virtualization hosts, backup systems, AD CS CAs, Entra Connect under some configurations, privileged management platforms, and accounts with replication or GPO control. Protecting only the DC operating system while leaving these dependencies exposed is an incomplete security model.

---
# Hybrid Identity, AD CS, LAPS, Service Accounts, and RODCs

*Questions 161-180 cover identity synchronization, federation, certificate services, managed credentials, and branch-office domain controllers.*

## 161. What is Microsoft Entra Connect Sync, and where does it fit in a hybrid identity design?

**Level:** Intermediate

**Answer:** Microsoft Entra Connect Sync synchronizes selected identities and attributes from on-premises Active Directory Domain Services to Microsoft Entra ID. It is not a domain controller, does not copy the AD database into the cloud, and does not make Entra ID an AD DS replica. The synchronization engine reads configured directories, applies join and transformation rules in its metaverse, and exports permitted changes to connected directories. A strong design defines authoritative sources, OU and attribute scope, source anchors, writeback requirements, service-account permissions, staging-server strategy, monitoring, and recovery procedures. Treat the synchronization server as part of the privileged identity control plane because it holds credentials and permissions that can materially affect both environments. Do not install unrelated applications on it, restrict interactive access, protect its database and configuration, and monitor connector-account and synchronization-rule changes.

## 162. How do Password Hash Synchronization, Pass-Through Authentication, and federation differ?

**Level:** Intermediate

**Answer:** Password Hash Synchronization, or PHS, synchronizes a derived hash of the on-premises password hash to Entra ID, allowing cloud authentication to continue even when on-premises authentication infrastructure is unavailable. Pass-Through Authentication, or PTA, validates cloud sign-ins through on-premises agents and therefore depends on agent and network availability for authentication. Federation redirects authentication to a federation service such as AD FS, giving the organization more control over the authentication flow but adding servers, certificates, dependencies, and operational risk. The choice should be driven by security requirements, availability objectives, regulatory constraints, legacy application needs, and operational maturity. PHS is generally the simplest and most resilient option; PTA and federation require explicit high-availability and disaster-recovery designs. Seamless SSO is a separate user-experience feature and should not be confused with the primary authentication method.

## 163. What security risks should be considered with Password Hash Synchronization?

**Level:** Advanced

**Answer:** PHS does not send a user's plaintext password to Entra ID. The synchronization process obtains the on-premises NT hash through replication APIs, applies additional processing and salting, and synchronizes a nonreversible derivative used for cloud authentication. The principal risks are therefore around privileged synchronization infrastructure, excessive directory-replication rights, connector-account compromise, tampering with synchronization rules, and poor monitoring. Restrict replication permissions to the required connector account, use a dedicated hardened server, limit administrators, protect backups, monitor privileged changes, and maintain an emergency recovery plan. Also understand scope: excluding an account from synchronization does not automatically remove every cloud object or credential already created. Security review should cover source-anchor selection, accidental matching, privileged-account synchronization policy, password writeback, and whether cloud-only break-glass accounts are independently protected and tested.

## 164. Why is an Entra Connect server often treated as a Tier 0 or control-plane system?

**Level:** Advanced

**Answer:** Depending on configuration, the server and its connector accounts can read sensitive directory attributes, perform directory replication operations, modify cloud identities, execute synchronization rules, and support writeback features. Compromise could therefore create or alter identities, synchronize malicious attributes, expose credential-derived material, or bridge an intrusion between on-premises AD and Entra ID. Place the server in a hardened management boundary, restrict logon rights, use dedicated administrators, apply rapid patching, monitor local and directory changes, and avoid browsing, email, or general administration from it. Maintain a tested staging server or documented rebuild procedure, but do not assume two active synchronization servers can export simultaneously. Review connector permissions after enabling or disabling features because historical privileges may remain broader than current requirements.

## 165. What is AD FS, and what are its major security dependencies?

**Level:** Intermediate

**Answer:** Active Directory Federation Services is a Security Token Service that authenticates users and issues signed claims-based tokens to relying parties. Its trust model depends on the integrity of the federation servers, service account, token-signing and token-decrypting certificates, relying-party configuration, claims rules, web application proxies, DNS, TLS, and AD DS. A stolen token-signing key can enable forged tokens until the trust and keys are remediated, so certificate protection and rollover are critical. Harden federation servers as privileged assets, minimize local administrators, use managed service accounts where supported, monitor configuration and certificate exports, restrict administrative endpoints, and keep proxies separated from internal federation servers. Organizations migrating away from AD FS should inventory every relying party and authentication requirement rather than simply removing servers after enabling cloud authentication.

## 166. What is the difference between an enterprise CA and a standalone CA in AD CS?

**Level:** Intermediate

**Answer:** An enterprise certification authority integrates with AD DS, uses certificate templates, can publish certificates and revocation information into the directory, and can support autoenrollment. A standalone CA does not require AD DS integration and generally processes requests using local policy rather than enterprise templates. Enterprise CAs are convenient but become highly privileged because certificates can authenticate users and computers, sign code, encrypt data, or establish trust. Standalone offline root CAs are commonly used in a two-tier PKI, with one or more online enterprise issuing CAs beneath them. The design should define trust boundaries, issuance policies, hardware protection, administrator separation, certificate lifetimes, revocation availability, audit retention, disaster recovery, and key-compromise procedures. “The CA is not a domain controller” does not mean it is lower impact; a compromised enterprise CA can undermine the identity system.

## 167. What is a certificate template?

**Level:** Intermediate

**Answer:** A certificate template is an AD object that defines how an enterprise CA may issue a class of certificates. It controls intended purposes, subject-name construction, cryptographic settings, validity, renewal, enrollment permissions, approval requirements, authorized signatures, issuance policies, and whether private keys may be exported. A CA must also be configured to issue the template. Security review must examine both template permissions and CA policy because an apparently harmless permission can become dangerous when a requester may supply an arbitrary subject alternative name, obtain an authentication-capable certificate, alter the template, act as an enrollment agent, or bypass approval. Remove unused templates from issuance, restrict owners and write permissions, avoid broad enrollment for privileged authentication templates, and monitor template and CA configuration changes.

## 168. How does certificate autoenrollment work?

**Level:** Intermediate

**Answer:** Autoenrollment uses Group Policy and enterprise CA integration to let eligible users or computers automatically request, renew, and manage certificates based on certificate-template permissions and settings. During policy processing, the client evaluates templates, existing certificates, renewal thresholds, and CA availability. Troubleshooting requires checking GPO scope, template issuance, Read and Enroll or Autoenroll permissions, DNS and RPC connectivity to the CA, certificate-services event logs, enrollment-policy endpoints, time, and the certificate store. Run `gpresult`, inspect `certlm.msc` or `certmgr.msc`, and use `certutil` diagnostic commands appropriate to the case. Do not solve enrollment failures by broadly granting Enroll or Full Control; that can turn an availability issue into a privilege-escalation path.

## 169. How can certificates be used for Active Directory authentication?

**Level:** Advanced

**Answer:** AD can map a certificate to an account and use public-key authentication mechanisms such as PKINIT to obtain a Kerberos ticket. Smart-card logon and Windows Hello for Business certificate-trust deployments are examples. The certificate must chain to a trusted authority, contain suitable enhanced key usages and identity information, meet mapping rules, and pass validity and revocation checks. Because a valid authentication certificate may substitute for a password, issuance controls are identity controls. Protect CA keys, certificate templates, enrollment agents, request approval, private keys, and account mappings. During incident response, disabling an account may not be sufficient if certificates, refresh tokens, or cached tickets remain valid; revoke or invalidate the relevant credentials and verify how revocation is consumed by each relying service.

## 170. What are the common security classes of AD CS misconfiguration?

**Level:** Advanced

**Answer:** Common classes include templates that let low-privileged requesters supply another identity, templates with authentication purposes and weak enrollment controls, enrollment-agent abuse, writable templates or CA objects, dangerous CA-wide flags, overly privileged CA managers, web-enrollment endpoints exposed to NTLM relay, weak certificate mappings, and inadequate protection of CA private keys. The “ESC” labels used by security tools are useful shorthand, but risk depends on the exact template, CA, mapping behavior, requester rights, and available authentication paths. Defenders should inventory CAs and templates, identify who can enroll and who can modify policy, require secure transport and relay protections, remove obsolete enrollment endpoints, apply current patches, monitor certificate issuance and configuration changes, and test revocation and key-compromise procedures. Do not remediate solely by renaming a template or removing one enrollment permission without reviewing inherited and indirect rights.

## 171. What is Windows LAPS?

**Level:** Intermediate

**Answer:** Windows Local Administrator Password Solution manages a unique, random local administrator password for each joined device and backs it up to AD DS or Microsoft Entra ID according to policy. It reduces the lateral-movement risk created by a shared local administrator password. Windows LAPS is built into supported Windows versions and adds features beyond legacy Microsoft LAPS, including password encryption in AD, password history, post-authentication actions, automatic account management on newer systems, and support for Directory Services Repair Mode passwords on domain controllers. A secure deployment delegates password-read and reset rights narrowly, protects recovery workflows, audits reads, validates schema and policy, and prevents help-desk staff from receiving broad control over computer objects. LAPS does not eliminate the need to restrict local administrator use or protect privileged workstations.

## 172. How does Windows LAPS differ from legacy Microsoft LAPS?

**Level:** Intermediate

**Answer:** Legacy Microsoft LAPS used a separately installed client-side extension and stored the managed password in attributes such as `ms-Mcs-AdmPwd`. Windows LAPS is integrated into supported Windows operating systems and uses newer attributes and policy settings. It can encrypt passwords in AD DS, maintain password history, back up to Entra ID, manage DSRM credentials, and perform post-authentication actions. Windows LAPS includes emulation modes for migration, but legacy and new policies must be planned carefully to avoid conflicting ownership of the account or password. Migration should inventory clients, schema state, GPOs, readers, scripts, management tools, and operational dependencies. Remove obsolete broad permissions after transition and verify that monitoring covers both old and new password attributes until legacy LAPS is fully retired.

## 173. How should access to LAPS passwords be secured and audited?

**Level:** Advanced

**Answer:** Delegate password retrieval only to roles that genuinely need it, preferably through controlled just-in-time workflows. Review extended rights on computer OUs, inherited permissions, groups allowed to decrypt encrypted passwords, and accounts allowed to reset expiration. Separate the ability to read a password from the ability to modify computer objects or policy. Enable directory-service auditing for sensitive attribute access where operationally feasible, collect Windows LAPS operational events, and monitor unusual volume, off-hours retrieval, or access to high-value systems. Protect management consoles and transcripts because the password can leak after a legitimate read. Configure rapid rotation after use and define incident handling for suspected disclosure. Encryption in AD reduces exposure to generic directory readers, but it does not protect against a compromised authorized decryptor or Tier 0 administrator.

## 174. What is a standalone Managed Service Account?

**Level:** Intermediate

**Answer:** A standalone Managed Service Account, or sMSA, is a domain account whose password is automatically managed for a service running on one computer. It reduces manual password handling and supports automatic SPN management in suitable configurations. Because it is tied to a single host, it is not appropriate for a load-balanced service running on multiple servers. The service and application must support using the account without an interactively supplied password. Administrators should still limit logon rights, service permissions, local privileges, and access to the host. For multi-host services, a group Managed Service Account is generally the better fit.

## 175. What is a group Managed Service Account?

**Level:** Intermediate

**Answer:** A group Managed Service Account, or gMSA, is a domain service account with an automatically managed password that can be retrieved by a defined set of authorized computers. It supports services, scheduled tasks, and application pools across multiple hosts when the application supports managed accounts. The Key Distribution Service derives password material from a KDS root key, and AD stores authorization information controlling which principals may retrieve it. Security depends heavily on the `PrincipalsAllowedToRetrieveManagedPassword` scope: compromising an authorized host may expose the gMSA credential. Use a separate gMSA per service boundary, minimize retrieval hosts, avoid interactive logon, monitor configuration changes, and remove retired hosts promptly. A gMSA solves password rotation, not excessive service privileges.

## 176. What is a delegated Managed Service Account in Windows Server 2025?

**Level:** Expert

**Answer:** A delegated Managed Service Account, or dMSA, is a Windows Server 2025 capability designed to replace traditional service accounts while tying credential use more closely to authorized machine identities. It provides managed credentials and a migration path intended to reduce the risks of static service-account passwords and credential theft. Deployment requires compatible domain controllers and clients, current documentation review, application testing, and careful migration of SPNs, rights, dependencies, and rollback procedures. Treat dMSA adoption as an identity-engineering project rather than a simple account rename. Inventory every service, determine where it runs, identify its required network access, remove interactive logon, and monitor any fallback to the legacy account. Continue using gMSAs where dMSA prerequisites or application support are not met.

## 177. What are the core service-account security practices?

**Level:** Intermediate

**Answer:** Prefer gMSA or dMSA where supported. Create a separate identity per service or trust boundary, grant only required local and network rights, deny interactive and remote interactive logon, restrict where the account may authenticate, and use strong automatically rotated credentials. Document the owner, application, hosts, SPNs, dependencies, recovery process, and retirement date. Do not place service accounts in Domain Admins or other broad groups to solve access problems. Monitor abnormal logon types, new hosts, SPN changes, group membership, delegation settings, and password resets. For unavoidable traditional accounts, use a vault, long random passwords, controlled rotation, and tested application restart procedures. An account with “password never expires” and an unknown owner is technical debt and a likely attack path.

## 178. What is the KDS root key, and why is it required for gMSAs?

**Level:** Advanced

**Answer:** The Key Distribution Service root key provides secret material from which domain controllers derive gMSA passwords. The key must replicate to the DCs that will service password retrieval. In production, administrators commonly allow time for replication before creating and using gMSAs; forcing immediate effective time should be limited to controlled lab scenarios because another DC may not yet have the key. Verify replication, domain functional prerequisites, host authorization, and service support before rollout. Protect KDS configuration as Tier 0 data. Recreating keys casually is not a routine troubleshooting step and can disrupt managed accounts; diagnose replication and authorization first.

## 179. What is the RODC Password Replication Policy?

**Level:** Advanced

**Answer:** The Password Replication Policy controls which account credentials an RODC may cache. The Allowed RODC Password Replication Group can permit caching, while the Denied RODC Password Replication Group and explicit denials prevent high-value credentials from being stored. Deny takes precedence. Design the policy around the branch's users and computers, not convenience. Prepopulate only required credentials, ensure privileged accounts never authenticate to the RODC site where possible, and review the set of accounts whose passwords were actually cached. An RODC still holds a read-only directory copy, its own machine secret, Kerberos material scoped to that RODC, DNS data if installed, and local operating-system state; it is risk-reduced, not risk-free.

## 180. How should an organization respond to a stolen or compromised RODC?

**Level:** Expert

**Answer:** Isolate and disable the RODC account, preserve evidence if the event is malicious, and use the RODC account's password-replication information to identify credentials that were cached or exposed. Reset the RODC computer account and the passwords of affected user and computer accounts according to a prioritized plan. Remove stale DNS and metadata, review replication and authentication logs, examine whether branch administrator credentials or network secrets were present, and rebuild rather than trust the recovered server. Because each RODC has a separate `krbtgt_<number>` account, exposure is more contained than compromise of the writable domain `krbtgt`, but cached credentials can still enable lateral movement. Validate the branch's physical controls, PRP design, administrator role separation, BitLocker configuration, and monitoring before redeployment.

---
# AD Security, Attack Paths, Detection, and Hardening

*Questions 181-200 explain common identity attack paths from a defensive interview perspective: prerequisite, impact, evidence, and remediation.*

## 181. What is the Active Directory attack surface?

**Level:** Intermediate

**Answer:** The AD attack surface includes more than domain controllers. It includes users and computers, privileged groups, service accounts, delegated ACLs, Group Policy, trusts, DNS, AD CS, federation, synchronization servers, management systems, virtualization hosts, backup infrastructure, endpoint credential exposure, and protocols such as Kerberos, NTLM, LDAP, SMB, and RPC. Attackers often progress through a chain of individually ordinary permissions rather than one critical vulnerability. A useful assessment maps identities to sessions, local administration, group membership, object-control rights, delegation, certificate enrollment, replication rights, and cross-domain reachability. Prioritize paths that lead to Tier 0 and distinguish theoretical graph edges from operationally exploitable paths. Remediation should remove unnecessary privilege, reduce credential exposure, harden protocols, isolate administration, and continuously verify that attack paths have not reappeared.

## 182. What is BloodHound, and how should its results be interpreted?

**Level:** Intermediate

**Answer:** BloodHound models relationships among AD principals, computers, groups, sessions, ACLs, local administrative rights, delegation, trusts, and other control paths as a graph. Defenders use it to identify shortest paths to high-value assets, hidden privilege, and excessive delegation. A graph edge means a documented relationship or potential capability, not guaranteed compromise. Validate collection freshness, tool version, data completeness, endpoint visibility, nested groups, and environmental controls before accepting a path. Remediate the root control relationship rather than merely hiding the final node. Run collection under authorized accounts, protect the data because it is an attack map, and compare snapshots over time to detect privilege drift. Similar analysis can be performed with Microsoft Defender for Identity posture assessments and native directory reviews.

## 183. What is DCSync?

**Level:** Advanced

**Answer:** DCSync is abuse of the directory replication protocol by an account that has sufficient replication rights, allowing it to request password-derived data and other replicated secrets as though it were a domain controller. It does not require interactive logon to a DC. Rights commonly associated with the capability include Replicating Directory Changes and Replicating Directory Changes All, with additional rights relevant to certain filtered attributes. Defenders should minimize these permissions, inventory all non-DC principals that hold them, monitor replication requests from systems that are not DCs, and investigate privileged-directory changes. Response requires containing the principal and host, determining which secrets were accessed, rotating exposed privileged and service credentials, considering `krbtgt` rotation, and checking for persistence. Simply resetting the attacker's password may not invalidate tickets or copied secrets.

## 184. What is DCShadow?

**Level:** Expert

**Answer:** DCShadow is a technique that registers attacker-controlled components as a domain controller and submits changes through replication interfaces, potentially bypassing monitoring focused only on normal LDAP modifications. It requires substantial directory and infrastructure privilege, so its presence usually indicates a severe compromise. Defenders should monitor creation or modification of server, NTDS Settings, service principal name, and replication-related objects; detect unexpected replication partners; restrict privileged administration; and collect domain-controller, network, and configuration-partition telemetry. During response, isolate the involved systems, compare replication metadata across DCs, review object versions and originating invocation IDs, remove unauthorized metadata, and search for resulting persistence such as ACL, group, SIDHistory, or AdminSDHolder changes. Restore trust in the identity control plane before returning systems to production.

## 185. What is a Golden Ticket, and what is the correct response?

**Level:** Advanced

**Answer:** A Golden Ticket is a forged Kerberos Ticket Granting Ticket created using the secret of the domain's `krbtgt` account. It can contain fabricated authorization data and remain usable within ticket validity and persistence constraints even after an ordinary user password reset. Detection relies on anomalies such as impossible account or group combinations, unusual ticket properties, authentication from unexpected systems, inconsistent lifetimes, and correlated endpoint or network evidence; no single event always proves forgery. Response requires treating the domain as deeply compromised, containing attacker access, removing persistence, resetting privileged and service credentials, and resetting the `krbtgt` account twice with adequate replication and validation between resets. A rushed rotation before containment can disrupt services while leaving the attacker able to reacquire secrets.

## 186. What is Kerberoasting, and how is it mitigated?

**Level:** Advanced

**Answer:** Kerberoasting abuses the legitimate ability of an authenticated user to request a service ticket for an SPN. Part of that ticket is encrypted with a key derived from the service account credential, allowing offline password guessing. The request itself is not proof of malicious activity because applications routinely request service tickets. Reduce risk by using gMSAs or dMSAs, long random passwords for traditional service accounts, AES-capable accounts, least privilege, restricted logon, and retirement of unused SPNs. Monitor unusual volume, broad requests across many SPNs, weak encryption use, and activity from endpoints or users that do not normally access those services. If a service account may have been cracked, rotate it safely, review where it logged on and what it could access, and investigate lateral movement rather than treating rotation as the complete response.

## 187. What is AS-REP roasting, and how is it prevented?

**Level:** Advanced

**Answer:** AS-REP roasting applies to accounts configured not to require Kerberos preauthentication. An attacker can request an authentication response for such an account without proving knowledge of its password and then attempt offline password guessing against encrypted response data. Prevent it by ensuring preauthentication is required unless a documented legacy exception exists, using strong managed credentials for any exception, and monitoring changes to the relevant `userAccountControl` flag. Inventory accounts with “Do not require Kerberos preauthentication,” identify owners and dependencies, and remove obsolete exceptions. During investigation, correlate authentication-service requests, source systems, account configuration changes, password age, and subsequent logons. The mere existence of an exception is a vulnerability condition; confirmed cracking requires additional evidence.

## 188. What is NTLM relay?

**Level:** Advanced

**Answer:** NTLM relay forwards a captured or coerced NTLM authentication exchange to another service that accepts it, allowing the attacker to act with the victim's privileges without learning the plaintext password. Success depends on protocol and target conditions such as missing SMB signing, weak LDAP signing or channel binding, exposed web enrollment, lack of Extended Protection, and the ability to induce authentication. Mitigation is layered: reduce or block NTLM where possible, require SMB signing, require LDAP signing and channel binding according to compatibility testing, enable Extended Protection on applicable services, disable unnecessary name-resolution fallbacks and coercible services, segment administration, and restrict outbound authentication from high-value systems. Detection should correlate inbound authentication, source and destination mismatch, unusual machine-account actions, certificate issuance, and configuration changes resulting from the relay.

## 189. Why are LDAP signing, LDAP channel binding, and SMB signing important?

**Level:** Advanced

**Answer:** LDAP signing protects integrity and authenticates the LDAP session so an intermediary cannot silently modify signed traffic. LDAP channel binding binds authentication to the TLS channel and helps prevent relaying credentials into a different protected session. SMB signing provides message integrity and makes common SMB relay attacks substantially harder. These controls are related but not interchangeable, and enforcement can break legacy applications, devices, or old operating systems. Begin with auditing and inventory, identify unsigned binds and incompatible clients, remediate them, then phase enforcement with rollback and exception governance. Use LDAPS or StartTLS correctly, but remember that TLS alone does not automatically guarantee the desired authentication binding. Monitor policy drift after rollout and do not leave “temporary” compatibility exceptions without owners and expiry dates.

## 190. What is password spraying, and how does it differ from brute force?

**Level:** Intermediate

**Answer:** Password spraying tests one or a small number of likely passwords across many accounts, often slowly enough to avoid per-account lockout thresholds. Traditional brute force tests many passwords against one account. Spraying frequently targets remote access, cloud sign-in, Kerberos, LDAP, SMB, or web authentication and may use valid usernames obtained from public or directory sources. Defenses include phishing-resistant MFA where supported, banned-password protection, strong password policy, elimination of shared and stale accounts, smart lockout, network restrictions, and detection based on one source or infrastructure pattern touching many accounts. Investigators should aggregate failures by source, password pattern where available, protocol, time window, and targeted population. Do not automatically lock every account during response; indiscriminate action can cause a second denial-of-service incident.

## 191. How can Group Policy be abused by an attacker?

**Level:** Advanced

**Answer:** Anyone who can create, edit, link, or take ownership of a GPO, or modify its files in SYSVOL, may be able to execute scripts, change security settings, deploy scheduled tasks or software, weaken defenses, or grant local privilege on affected systems. The impact depends on the link location, security filtering, WMI filters, inheritance, and whether computer or user settings are controlled. Defenders should review GPO and OU ACLs, separate GPO creation from linking, protect SYSVOL, monitor directory and file changes, maintain version-controlled backups, and restrict privileged GPO management to hardened workstations. During response, compare the Group Policy Container and Template versions, inspect changed settings and files, identify affected scope, collect endpoint execution evidence, restore a known-good policy, and remove the underlying delegated right.

## 192. How are AD ACLs abused for privilege escalation?

**Level:** Advanced

**Answer:** An attacker may not need group membership if an ACL grants control over a user, group, computer, GPO, OU, domain object, certificate template, or other principal. Rights such as GenericAll, GenericWrite, WriteDACL, WriteOwner, password reset, group membership modification, SPN modification, delegation configuration, or replication rights can form privilege paths. Effective access may come through nested groups and inheritance, making it easy to miss in GUI reviews. Defenders should inventory sensitive-object ACLs, compare them with an approved baseline, remove orphaned SIDs and broad inherited rights, and monitor ownership and DACL changes. In an incident, identify the exact ACE, who added it, when it replicated, what actions it enabled, and whether the attacker used those capabilities before removing the ACE.

## 193. What is AdminSDHolder-based persistence?

**Level:** Advanced

**Answer:** AdminSDHolder is a protected directory object whose security descriptor is periodically applied by the Security Descriptor Propagator to accounts and groups considered protected. An attacker with sufficient rights can add a malicious ACE to AdminSDHolder and have that permission propagated to many privileged objects. Another common issue is that accounts formerly in protected groups may retain `adminCount=1` and disabled inheritance, causing delegated administration to fail and leaving unexpected permissions. Monitor AdminSDHolder's DACL and owner, privileged-group changes, `adminCount`, and inheritance state. Remediation requires removing unauthorized rights from AdminSDHolder, correcting affected objects, and investigating the actor and originating DC through audit and replication metadata. Do not blindly clear `adminCount` on active protected accounts.

## 194. What is SIDHistory, and how can it be abused?

**Level:** Advanced

**Answer:** `sIDHistory` stores previous security identifiers so migrated users or groups can continue accessing resources that still reference old SIDs. It is useful during controlled migrations but can also grant hidden access if a privileged SID is inserted or retained. Within a forest, trust and authorization behavior can make malicious SID history especially dangerous. Limit who can perform migrations and write the attribute, enable appropriate trust protections such as SID filtering where applicable, monitor changes, and remove historical SIDs after resource ACLs are remediated. During investigation, compare values with migration records, identify the originating DC and administrative actor, review token group membership and resource access, and check whether trust settings were weakened. A SIDHistory value is not malicious by itself; undocumented privileged values are the concern.

## 195. How can AD CS become a domain-compromise path?

**Level:** Advanced

**Answer:** Certificates can authenticate as users or computers, so a CA or template that permits a low-privileged requester to obtain a certificate representing a privileged identity can bypass password controls. Paths can arise through subject-name control, authentication-capable templates, enrollment-agent permissions, writable templates, CA management rights, relayable enrollment endpoints, weak mappings, or stolen CA keys. Treat AD CS as Tier 0, maintain an inventory of CAs and issued templates, review template and CA ACLs, require strong request identity and approval where appropriate, secure web and RPC enrollment, patch mapping vulnerabilities, and monitor issuance. Incident response may require revocation, disabling templates, CA configuration rollback, account and certificate investigation, trust-store changes, or full CA key replacement depending on what was compromised.

## 196. What are the security differences among unconstrained, constrained, and resource-based constrained delegation?

**Level:** Advanced

**Answer:** Unconstrained delegation can place reusable user Kerberos material on the delegated server and is high risk, especially when privileged users connect. Traditional constrained delegation allows an account to delegate to specified services and is configured on the delegating account. Resource-based constrained delegation, or RBCD, is configured on the target resource through `msDS-AllowedToActOnBehalfOfOtherIdentity`, allowing the resource owner to choose which principals may delegate to it. Each model can be abused if the delegating identity, target object, or relevant ACL is compromised. Prefer eliminating unconstrained delegation, mark sensitive accounts as nondelegable, use Protected Users where compatible, restrict who can modify computer objects, monitor delegation attributes, and separate service identities. Validate actual application requirements before removing delegation because incorrect changes can break multi-tier authentication.

## 197. Why are LAPS and gMSA read permissions security-sensitive?

**Level:** Advanced

**Answer:** Reading a LAPS password can provide local administrator access to a device. Retrieving a gMSA managed password can allow authentication as the service account from an authorized or compromised context. Therefore, permissions that appear to be “read-only directory access” may create executable privilege. Review extended rights, confidential-attribute access, encrypted LAPS decryptors, gMSA retrieval principals, nested groups, and inherited OU permissions. Minimize readers, separate operational roles, use just-in-time access, monitor retrieval and permission changes, and treat authorized reader endpoints as privileged. During incident response, determine which secrets were read, rotate them, inspect affected hosts and service access, and remove the root permission. Rotating without containing an authorized compromised reader permits immediate reacquisition.

## 198. Which Windows events are especially useful for Active Directory security monitoring?

**Level:** Intermediate

**Answer:** Useful events include successful and failed logons (`4624`, `4625`), explicit credentials (`4648`), special privileges (`4672`), account and group changes (`4720`-series, `4732`, `4756`, and related events), account lockout (`4740`), Kerberos ticket activity (`4768`, `4769`, `4771`), NTLM credential validation (`4776`), directory-service changes (`5136`, `5137`, `5139`, `5141`), sensitive directory access such as replication (`4662` when auditing is configured), share access, process creation (`4688`), policy changes, and log clearing (`1102`). Domain Controller, DFS Replication, DNS Server, AD CS, Windows LAPS, PowerShell, and Defender logs add context. Event IDs alone are not detections; configure the correct audit subcategories and SACLs, centralize logs, normalize fields, baseline legitimate administration, and correlate identity, endpoint, network, and cloud telemetry.

## 199. What is Microsoft Defender for Identity?

**Level:** Intermediate

**Answer:** Microsoft Defender for Identity is a cloud-based identity security service that analyzes signals from domain controllers and integrated identity infrastructure to detect suspicious identity activity, expose security posture weaknesses, and support investigation. Sensors observe relevant authentication and directory activity and enrich it with identity context. Effective deployment requires complete coverage of writable DCs and relevant AD FS or AD CS systems, sensor health monitoring, directory permissions, network and event configuration, and integration with the broader Microsoft Defender portal. Treat alerts as investigative leads rather than automatic proof. Tune known administrative systems, investigate the full attack timeline, and remediate posture findings such as risky delegation, exposed credentials, weak protocols, and excessive privilege. A sensor does not replace hardened configuration, centralized Windows auditing, or recovery readiness.

## 200. What should a prioritized Active Directory hardening roadmap include?

**Level:** Expert

**Answer:** First establish recovery: supported DCs, healthy replication and DNS, isolated tested system-state backups, and a forest recovery runbook. Next secure Tier 0: minimize privileged membership, use separate admin identities and hardened workstations, protect DCs, CAs, synchronization, hypervisors, backups, and management tools. Reduce credential exposure with Windows LAPS, gMSA or dMSA, Protected Users where compatible, authentication policies, and removal of privileged logons from lower-tier systems. Then eliminate known attack paths: excessive ACLs, dangerous delegation, stale trusts, risky certificate templates, shared service passwords, legacy protocols, unsigned LDAP or SMB, and unmanaged GPO rights. Improve visibility with advanced auditing, Defender for Identity or equivalent analytics, time synchronization, asset ownership, and alert-response procedures. Measure progress through recurring attack-path reviews, recovery tests, protocol-usage reduction, and privileged-access recertification.

---
# L2 Operations Scenarios

*Questions 201-220 test methodical administration and support under realistic operational pressure.*

## 201. A newly created user cannot sign in, but existing users can. How do you troubleshoot it?

**Level:** Intermediate Scenario

**Answer:** Confirm the exact username format, domain, workstation, error text, and whether the user can sign in against a specific DC. Check that the account is enabled, has a valid UPN and `sAMAccountName`, is within logon hours, is not expired, and is not subject to workstation restrictions. Verify the password was set correctly and whether “must change password” conflicts with the logon method. Use `Get-ADUser -Properties *`, inspect security events on the authenticating DC, and identify that DC with `nltest /dsgetdc:<domain>` or the client log. If the user was created recently, check replication with `repadmin /replsummary` and `repadmin /showrepl`. Also verify DNS and time. Avoid repeatedly resetting the password before determining whether the client is contacting a DC that has received the object.

## 202. A user changed a password at headquarters, but the old password still works at a branch. What do you check?

**Level:** Intermediate Scenario

**Answer:** Identify which DC processed the password change and which DC authenticates the branch user. Password changes receive urgent replication treatment, and failed authentication may be forwarded to the PDC Emulator, but network or replication failure can still create inconsistent behavior. Run `repadmin /showrepl`, `repadmin /replsummary`, and targeted replication checks between the involved DCs. Verify site and subnet mapping, DNS resolution, firewall paths, time, and the PDC Emulator's availability. Review `pwdLastSet` and replication metadata on multiple DCs. Cached interactive credentials can also allow an offline workstation logon and must be distinguished from online domain authentication. Fix the replication or site problem; do not normalize the condition by asking the user to maintain two passwords.

## 203. A user's account keeps locking every few minutes. How do you find the source?

**Level:** Intermediate Scenario

**Answer:** Establish the lockout time and the DC that recorded event `4740`; its caller-computer field often identifies the source. Query the PDC Emulator and other DCs, because lockout processing is coordinated around the PDC. Correlate `4625`, `4771`, `4776`, VPN, wireless, email, scheduled-task, service, IIS, mobile-device, and application logs. Common causes are saved credentials, disconnected RDP sessions, services, mapped drives, old phone passwords, and scripts. Check `badPwdCount` carefully because it is DC-specific and can change as authentication moves. Do not immediately raise the lockout threshold or disable the policy. If failures span many accounts or sources, treat it as possible password spraying and involve security operations.

## 204. A mapped drive GPO applies to everyone except one user. What is your troubleshooting sequence?

**Level:** Intermediate Scenario

**Answer:** Confirm the user, computer, target OU, and whether the mapping is a user or computer preference. Generate `gpresult /h report.html` in the affected session and compare it with a working user. Check GPO link, enabled sections, inheritance, enforced or blocked inheritance, security filtering, WMI filters, loopback processing, item-level targeting, group membership, and whether the user has Read and Apply Group Policy. Verify SYSVOL access and GPO version consistency, then inspect Group Policy operational events. For Group Policy Preferences, review action type, drive-letter conflicts, “run in logged-on user's security context,” and path permissions. Do not use repeated `gpupdate /force` as diagnosis; determine why the GPO is denied, filtered, unavailable, or failing during execution.

## 205. A workstation reports “The trust relationship between this workstation and the primary domain failed.” What happened, and how do you repair it?

**Level:** Intermediate Scenario

**Answer:** The computer's local machine-account password no longer matches the value in AD, or the client cannot contact a suitable DC and reports a misleading trust error. First verify DNS, time, network access, duplicate computer names, restored snapshots, and the secure channel with `Test-ComputerSecureChannel -Verbose` or `nltest /sc_verify:<domain>`. If connectivity is sound, repair the secure channel with an authorized method such as `Test-ComputerSecureChannel -Repair` or `Reset-ComputerMachinePassword`, using credentials that may reset the computer account. Rejoining the domain is a fallback, not the first step, because it can alter local profiles and application state. Investigate repeated failures for imaging, snapshot rollback, cloning, or duplicate-account processes.

## 206. A server cannot join the domain. Which checks do you perform before blaming permissions?

**Level:** Intermediate Scenario

**Answer:** Verify the server uses only internal AD-aware DNS servers and can resolve the domain's SRV records, such as `_ldap._tcp.dc._msdcs.<forest-root>`. Confirm forward and reverse resolution, time synchronization, reachability to required ports, and that the intended domain name is correct. Use `nltest /dsgetdc:<domain>`, `Test-NetConnection`, `nslookup`, and the join log in `%windir%\debug\NetSetup.log`. Check for an existing computer object with restrictive ACLs, quota or delegated join rights, duplicate names, unsupported encryption or SMB settings, and firewall inspection. Validate the joining credential only after infrastructure checks. The error code and NetSetup log usually identify whether the failure is discovery, authentication, account creation, secure-channel establishment, or policy.

## 207. Users at one branch experience very slow logons. How do you isolate the cause?

**Level:** Advanced Scenario

**Answer:** Measure where time is spent rather than treating “logon” as one action. Confirm the branch subnet is mapped to the correct AD site and which DC and Global Catalog clients use. Review DNS response, WAN latency and loss, time, DC Locator, replication, and authentication events. Compare a clean user and workstation with an affected one. Use Group Policy operational logs and `gpresult` to identify slow scripts, software installation, folder redirection, drive mappings, WMI filters, or synchronous processing. Check profile size, roaming-profile or FSLogix dependencies, certificate enrollment, printer discovery, and unavailable UNC paths. Validate branch DC health with `dcdiag` and `repadmin`. A new local DC will not fix a slow logon caused by a dead script path or oversized profile.

## 208. A legacy application says “LDAP bind failed” after security hardening. How do you troubleshoot without weakening the whole domain?

**Level:** Advanced Scenario

**Answer:** Capture the application host, bind type, target hostname, port, TLS behavior, service account, and exact error. Check Directory Service events for unsigned or rejected binds and confirm whether the change involved LDAP signing, channel binding, TLS versions, certificate trust, or NTLM restrictions. Test from the application host with a supported LDAP client and inspect the certificate chain and name. Upgrade or reconfigure the application to use SASL signing or LDAPS/StartTLS with proper certificate validation. If a temporary exception is unavoidable, scope it to the specific system or service with an owner, compensating controls, monitoring, and expiry date. Do not globally disable LDAP signing or channel binding to restore one undocumented application.

## 209. A Windows service fails after its account password was rotated. What should the administrator examine?

**Level:** Intermediate Scenario

**Answer:** Determine where the account is used before changing it again. Inventory services, scheduled tasks, IIS application pools, database connections, scripts, SPNs, and dependent servers. Check service-control and application logs for logon failure, account lockout, or access denied. Confirm the new credential was updated in every managed location, the account is not expired or disabled, and it still has “log on as a service” and resource permissions. Review SPNs for duplicate or missing registrations if Kerberos is involved. Restore service using a controlled credential rollback only when justified, then migrate suitable workloads to a gMSA or dMSA. The post-incident action is dependency documentation and automated rotation testing, not making the password nonexpiring.

## 210. A user was added to a file-access group but still receives Access Denied. What do you check?

**Level:** Intermediate Scenario

**Answer:** Confirm the change replicated to the user's authenticating DC and that the user has obtained a new access token; existing sessions do not automatically gain new group SIDs. Have the user sign out and sign in, or purge only the relevant Kerberos tickets when appropriate. Verify nested group scope and the AGDLP or AGUDLP design, then evaluate both share and NTFS permissions, explicit denies, ownership, dynamic access controls, and the exact path. Use `whoami /groups`, `klist`, effective-access tools, and access logs. If a universal group changed, check Global Catalog availability and replication. Do not keep nesting the user into additional privileged groups until access happens; identify the missing or denied authorization edge.

## 211. The help desk must reset passwords but must not administer privileged accounts. How do you delegate this safely?

**Level:** Intermediate Scenario

**Answer:** Place ordinary user accounts in controlled OUs and delegate the minimum required rights to a dedicated help-desk group using the Delegation of Control Wizard or explicit ACLs. Typical rights are reset password, force password change at next logon, unlock account, and read limited attributes. Keep privileged and service accounts in separate protected OUs with no inherited help-desk delegation. Test effective permissions using representative accounts and confirm AdminSDHolder-protected users behave as expected. Use separate named admin accounts, MFA and privileged workstations where possible, approval for sensitive resets, and logging of reset and group-change events. Periodically recertify membership and OU ACLs. Do not delegate Full Control over the OU merely because the wizard's default task does not cover one edge case.

## 212. How do you clean up stale computer accounts without breaking active systems?

**Level:** Intermediate Scenario

**Answer:** Use multiple signals: `lastLogonTimestamp`, per-DC `lastLogon` when higher precision is required, password age, DNS and DHCP records, endpoint-management inventory, vulnerability-scanner data, and business ownership. Define a conservative inactivity threshold and exclusions for offline, seasonal, disaster-recovery, kiosk, or embedded systems. First move candidates into a quarantine OU with restrictive but reversible policy or disable them, notify owners, monitor authentication failures, and retain a rollback window. Then delete according to retention policy and use Recycle Bin for recovery where enabled. Remove associated DNS, certificates, management records, and group memberships. A single old timestamp is not sufficient evidence because replication granularity, imaging, and long-offline devices can mislead.

## 213. How would you provision hundreds of users reliably?

**Level:** Intermediate Scenario

**Answer:** Start with an approved data source and schema: unique identifier, legal name, UPN, department, manager, location, employment dates, license or entitlement data, and owner. Validate uniqueness and naming rules before any write. Use idempotent PowerShell or an identity-governance workflow that can preview changes, log each operation, handle errors, and rerun safely. Create users disabled or with controlled initial credentials, place them in the correct OU, assign role-based groups rather than direct resource permissions, and enforce joiner-mover-leaver governance. Protect input files because they contain personal data. Test in a nonproduction OU and sample the results. Avoid scripts that silently continue after duplicate UPNs, failed manager references, or partial group assignment.

## 214. What is a safe process for decommissioning a domain controller?

**Level:** Advanced Scenario

**Answer:** Verify the DC is not the only DNS, Global Catalog, time, DHCP authorization, CA, application, or site dependency and does not hold FSMO roles that must move. Confirm healthy inbound and outbound replication, SYSVOL, backups, and replacement capacity. Update clients, network devices, applications, monitoring, and static DNS references. Demote gracefully using supported Server Manager or PowerShell, allowing metadata and DNS cleanup to occur. Then verify Sites and Services, DNS records, replication topology, DFSR membership, monitoring, and asset records. Securely erase or repurpose the server only after confirming no rollback is required. If graceful demotion fails, perform forced removal and metadata cleanup under a documented recovery procedure.

## 215. A new office is opening. How do you create the AD site design?

**Level:** Intermediate Scenario

**Answer:** Gather IP subnets, WAN paths, bandwidth, latency, business hours, expected users and devices, local-service requirements, and failure tolerance. Create the AD site and map every routed client subnet to it. Define site links reflecting actual network connectivity, costs, schedules, and replication intervals rather than the organizational chart. Decide whether the office needs a writable DC, RODC, DNS, or no local DC based on population, WAN reliability, physical security, and recovery objectives. Validate DC Locator from representative clients and monitor replication after deployment. Coordinate with network teams so new subnets are added during change management; an unmapped subnet can cause clients to select remote DCs even when a local DC exists.

## 216. You must move the PDC Emulator role. What do you verify before and after the transfer?

**Level:** Advanced Scenario

**Answer:** Select a healthy, well-connected, secured writable DC with reliable DNS and time connectivity. Confirm replication is clean, the target is advertising, and it can assume the domain's authoritative time role. Transfer rather than seize while the current holder is available. Use PowerShell or the appropriate MMC, then verify role ownership with `netdom query fsmo` or `Get-ADDomain`. Reconfigure and test the time hierarchy, monitor password-change and lockout behavior, check event logs, and confirm operations-master replication. Update monitoring and recovery documentation. A role transfer does not migrate unrelated services, DNS configuration, or external time settings automatically.

## 217. A domain controller's system drive is almost full. What is your response?

**Level:** Advanced Scenario

**Answer:** Treat low disk as an availability and integrity risk. Identify growth in logs, crash dumps, update caches, antivirus quarantine, SYSVOL, `ntds.dit`, ESE logs, or third-party agents. Verify backups and replication before deleting anything. Do not manually delete `ntds.dit`, ESE transaction logs, SYSVOL content, or unknown files to gain space. Free safe temporary space, extend the volume where supported, correct runaway logging, and move supported data paths only through documented procedures. Check Directory Service, DFSR, DNS, and system events for paused replication or database errors. If the DC is nonunique, rebuilding it may be safer than invasive database maintenance. Confirm healthy replication and backup after remediation.

## 218. A newly promoted DC does not publish SYSVOL or NETLOGON shares. How do you diagnose it?

**Level:** Advanced Scenario

**Answer:** Confirm the server completed promotion, rebooted, uses correct DNS, and has healthy AD replication. Check `net share`, DFS Replication event logs, `dcdiag /test:sysvolcheck /test:advertising`, `repadmin /showrepl`, and the DFSR state for the SYSVOL replicated folder. Determine whether initial synchronization is waiting on an unavailable or unhealthy upstream DC, whether the source is paused after an unexpected shutdown, or whether migration from FRS is incomplete. Fix AD replication, DNS, time, firewall, and DFSR health first. Do not use registry shortcuts that mark SYSVOL authoritative or bypass initial sync unless following a documented recovery procedure with a verified good source.

## 219. A security GPO applies to every laptop except one. How do you prove where the failure occurs?

**Level:** Intermediate Scenario

**Answer:** Confirm the laptop's computer object and OU, then generate computer-scope Resultant Set of Policy with administrative rights. In `gpresult /h`, inspect denied GPOs and reasons. Verify the client can locate a DC, read both the GPC in AD and GPT in SYSVOL, and resolve the domain DFS path. Compare the GPO's AD and SYSVOL version numbers, examine Group Policy operational events, and check security filtering, WMI filters, inheritance, slow-link behavior, and local policy conflicts. Run `gpupdate` once while collecting evidence. If only a particular setting fails, validate client-side extension support and policy precedence. Do not unlink and relink the GPO globally to repair one endpoint.

## 220. Clients repeatedly log “No logon servers available.” What categories of failure do you test?

**Level:** Advanced Scenario

**Answer:** Test client DNS configuration and SRV resolution, IP routing and firewall access, site and subnet mapping, DC service availability, time, secure channel, and whether a DC is advertising. Use `ipconfig /all`, `nltest /dsgetdc:<domain> /force`, `nltest /sc_verify:<domain>`, `dcdiag`, `repadmin`, and targeted port tests. Determine whether the issue is local to one client, subnet, site, DC, or authentication protocol. Review Netlogon, DNS Client, System, and DC logs. Cached logon success does not prove domain connectivity. Fix the dependency rather than adding public DNS as a secondary resolver or hard-coding a single remote DC, both of which often make AD discovery less reliable.

---
# L3 DNS, Replication, Authentication, and GPO Scenarios

*Questions 221-240 test root-cause analysis across distributed AD dependencies.*

## 221. Replication fails with RPC error 1722. How do you troubleshoot it?

**Level:** Advanced Scenario

**Answer:** Error 1722 means the RPC server is unavailable, but the root cause may be DNS, routing, firewall, service state, endpoint mapping, dynamic RPC ports, IPsec, or the target DC itself. Confirm both DCs resolve each other's current host and GUID-based CNAME records correctly. Test TCP 135 and the configured dynamic RPC range in both directions, then check `repadmin /showrepl`, `dcdiag /test:dns`, Netlogon, RPC, and Directory Service events. Verify time and secure channel. Compare the failure with other partners to identify whether the source, destination, or path is common. Packet capture can show failed name resolution or RPC endpoint negotiation. Do not “fix” it by opening all ports permanently without validating the required flows and network policy.

## 222. Replication returns error 8453, “Replication access was denied.” What are likely causes?

**Level:** Advanced Scenario

**Answer:** Check whether the involved DC computer accounts and NTDS Settings objects are intact, the secure channel works, and the DCs authenticate using Kerberos. Error 8453 can result from broken machine-account trust, duplicate or incorrect SPNs, time skew, DNS misresolution, damaged directory permissions, or a DC restored or cloned incorrectly. Review `repadmin /showrepl`, `dcdiag /test:checksecurityerror`, Kerberos events, `nltest /sc_verify`, and replication metadata. Confirm the servers are genuine DCs and no firewall or inspection device is forcing inappropriate authentication. Avoid granting broad replication rights to computer accounts as a workaround; normal DC replication permissions are established through protected directory objects and group membership. Repair the identity or topology fault.

## 223. Replication or logon fails with “The target principal name is incorrect.” What do you investigate?

**Level:** Expert Scenario

**Answer:** This usually indicates Kerberos cannot validate the service identity expected for the target. Check forward and reverse DNS, duplicate hostnames or IP addresses, duplicate SPNs, stale DC records, machine-account password mismatch, and whether a snapshot or image created two systems with related identity. Use `setspn -X`, `setspn -Q`, `nltest`, and `klist`; inspect the target's `servicePrincipalName` values and replication metadata. Confirm clients are connecting by the intended FQDN and not an unsupported alias. For DCs, validate GUID CNAME records and secure channels. Do not immediately disable Kerberos or force NTLM. Correct the duplicate identity, SPN, DNS, or machine-secret condition and then retest after ticket purge where appropriate.

## 224. How do you handle lingering objects detected by replication diagnostics?

**Level:** Expert Scenario

**Answer:** First stop treating the affected DC as a normal replication partner until scope is understood. Identify the source and destination naming context, how long the DC was offline, tombstone lifetime, strict replication consistency setting, and which replica contains the authoritative live state. Use `repadmin /removelingeringobjects` in advisory mode before removal and preserve command output. Remove lingering objects only against a verified reference DC and naming context. Investigate stale DNS, unsupported snapshot rollback, disabled replication, and monitoring gaps. A DC offline beyond tombstone lifetime is often safer to demote or rebuild than to rehabilitate. After cleanup, verify every partition, Global Catalog state, DNS application partitions, and replication convergence.

## 225. Replication is healthy during the day but consistently backlogs overnight. How do site-link settings factor in?

**Level:** Advanced Scenario

**Answer:** Review site-link schedules, intervals, costs, bridgehead selection, WAN maintenance windows, and whether backup or batch traffic saturates the link. A schedule may permit connections only during a narrow window, while change volume exceeds available bandwidth. Examine replication queue and largest delta over time rather than one snapshot. Confirm topology generation and connection objects, then correlate network telemetry and KCC events. Consider widening the schedule, reducing interval, improving capacity, or changing topology after modeling business impact. Compression and notification behavior differ between intersite and intrasite replication. Do not force full synchronization every night as a permanent fix; that can intensify congestion and mask an undersized or incorrectly scheduled topology.

## 226. Users receive `KRB_AP_ERR_MODIFIED` when accessing one service. What is the diagnostic logic?

**Level:** Advanced Scenario

**Answer:** The service ticket was encrypted for a key that the receiving service cannot use. Common causes are duplicate SPNs on different accounts, the SPN assigned to the wrong account, a load-balanced node using inconsistent service credentials, stale machine-account password, or a client resolving the name to the wrong host. Identify the exact SPN requested with Kerberos logs or `klist`, query it with `setspn -Q`, and check duplicates with `setspn -X`. Verify DNS, aliases, application-pool identity, service credential consistency, and account password replication. Correct the SPN and credential ownership, then purge tickets and restart only the affected service if required. Do not register the same SPN on every cluster node unless the service architecture explicitly supports it through one shared identity.

## 227. Access works when users connect by hostname but fails by IP address. Why?

**Level:** Intermediate Scenario

**Answer:** Kerberos normally identifies a service by SPN derived from a name, not an IP address. Connecting by IP often prevents the client from constructing a matching SPN and causes fallback to NTLM or failure if NTLM is restricted. The correct fix is to use a stable DNS name with a properly registered SPN and valid certificate where applicable. Check name resolution, application configuration, aliases, and whether the service supports Kerberos. Do not weaken NTLM policy solely to support hard-coded IP connections. If an appliance cannot use DNS names, document the exception, restrict its network path and account, and plan replacement or an application-layer solution.

## 228. Kerberos authentication fails across a site after a time-service change. How do you recover?

**Level:** Advanced Scenario

**Answer:** Measure actual offset on clients, member servers, DCs, and the PDC Emulator with `w32tm /query /status`, `/source`, and `/monitor`. Verify the domain hierarchy: domain members follow domain time, DCs follow the hierarchy, and the forest-root PDC Emulator uses reliable external sources. Check NTP reachability, GPO conflicts, hypervisor time integration, manual peer flags, and event logs. Correct the authoritative source and hierarchy rather than setting clocks manually on many systems. Large corrections may require careful service coordination. Once time converges, purge stale Kerberos tickets or restart affected services where justified. Investigate why monitoring did not catch offset before it exceeded authentication tolerance.

## 229. Clients in a site use a distant DC even though a local DC is healthy. What do you examine?

**Level:** Advanced Scenario

**Answer:** Check the client's IP address and whether its exact subnet is defined and associated with the correct AD site. Use `nltest /dsgetsite` and `nltest /dsgetdc:<domain> /force`. Confirm the local DC advertises the required capabilities, registers site-specific SRV records, hosts DNS and Global Catalog if needed, and is reachable on required ports. Review DNS server order, cached DC Locator information, site coverage, and whether the client is a VPN user whose assigned subnet maps elsewhere. Check Netlogon and DNS registration events on the DC. Do not hard-code a preferred DC in applications or clients as a substitute for correct site topology.

## 230. A DC is missing its SRV records. How do you restore them correctly?

**Level:** Advanced Scenario

**Answer:** Verify the DC points to authoritative AD-integrated DNS, its primary DNS suffix and domain membership are correct, and the Netlogon and DNS Client services are running. Check zone existence, dynamic-update policy, application-partition replication, permissions on existing records, and event logs. Run `dcdiag /test:dns`, inspect `%windir%\System32\Config\Netlogon.dns`, and trigger supported registration with `nltest /dsregdns` or restart Netlogon after correcting the cause. Register host records with `ipconfig /registerdns` as needed. Confirm the records appear on multiple DNS servers through replication. Manually adding a few SRV records is not a complete repair because site-specific and GUID records must remain accurate over time.

