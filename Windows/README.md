# Windows Security, Administration, and Digital Forensics Interview Questions and Answers

> A repo-ready Windows interview guide containing **250 original questions with detailed, technically validated answers** for Windows administrators, SOC analysts, DFIR investigators, endpoint-security engineers, incident responders, threat hunters, help-desk engineers, and cybersecurity candidates.

This guide is designed for `Interview-Hub/Windows`. It uses the user-supplied interview pages to identify frequently tested themes, but the technical answers are independently written and cross-checked against Microsoft Learn, Microsoft Sysinternals, NIST, CERT-In, CISA, MITRE ATT&CK, SWGDE, RFCs, and other standards-oriented sources.

> **Scope note:** Windows security and forensics change across versions, editions, policies, and enterprise tooling. Always validate commands and features in the exact Windows build and organizational environment. Legal and regulatory sections are interview preparation, not legal advice.

---

## How to Use This Guide

- **Questions 1-50:** Windows foundations, architecture, boot, processes, services, Registry, storage, and NTFS.
- **Questions 51-100:** Identity, authentication, authorization, permissions, networking, remote access, and management.
- **Questions 101-150:** Hardening, Defender, BitLocker, application control, auditing, Event Logs, PowerShell, Sysmon, and telemetry.
- **Questions 151-200:** Windows forensic artifacts, acquisition, memory, malware, timelines, and evidence interpretation.
- **Questions 201-250:** Incident response, threat hunting, troubleshooting, senior-level design, and scenario questions.

For a strong spoken interview answer, use four layers:

1. **Definition** — explain the concept precisely.
2. **Operational use** — show where it appears in real systems.
3. **Security or forensic value** — explain why it matters.
4. **Limitation** — state what the control, log, or artifact cannot prove by itself.

---

## Contents

1. [Windows Foundations and Architecture](#windows-foundations-and-architecture) — Questions 1-25  
2. [Processes, Services, Registry, Storage, and NTFS](#processes-services-registry-storage-and-ntfs) — Questions 26-50  
3. [Identity, Authentication, Authorization, and Permissions](#identity-authentication-authorization-and-permissions) — Questions 51-75  
4. [Windows Networking, Remote Access, and Management](#windows-networking-remote-access-and-management) — Questions 76-100  
5. [Windows Hardening and Endpoint Protection](#windows-hardening-and-endpoint-protection) — Questions 101-125  
6. [Auditing, Event Logs, PowerShell, Sysmon, and Monitoring](#auditing-event-logs-powershell-sysmon-and-monitoring) — Questions 126-150  
7. [Windows Forensic Artifacts](#windows-forensic-artifacts) — Questions 151-175  
8. [Acquisition, Memory, Malware, and Timeline Analysis](#acquisition-memory-malware-and-timeline-analysis) — Questions 176-200  
9. [Incident Response, Threat Hunting, and Troubleshooting](#incident-response-threat-hunting-and-troubleshooting) — Questions 201-225  
10. [Advanced Design and Scenario Questions](#advanced-design-and-scenario-questions) — Questions 226-250  
11. [Source and Validation Method](#source-and-validation-method)

---

# Windows Foundations and Architecture

## 1. What is Microsoft Windows?

**Answer:** Microsoft Windows is a family of operating systems that manages hardware, processes, memory, files, devices, users, security boundaries, networking, and application execution. Modern Windows uses the Windows NT architecture rather than the older MS-DOS architecture. In an enterprise, Windows is more than a graphical desktop: it provides security principals, access tokens, services, event logging, PowerShell, Group Policy processing, endpoint protection, encryption, remote management, and domain integration. A strong interview answer distinguishes the operating system from individual products such as Microsoft 365, Microsoft Defender for Endpoint, or Active Directory. Windows supplies local platform capabilities; cloud and enterprise services can extend those capabilities.

## 2. What is the difference between Windows client and Windows Server?

**Answer:** Windows client editions are optimized for interactive end-user workloads, while Windows Server editions are designed to host infrastructure and application roles at scale. Windows Server supports roles such as Active Directory Domain Services, DNS, DHCP, file services, Hyper-V, failover clustering, certificate services, and remote-access infrastructure. Client and server share many kernel and security concepts, but licensing, supported roles, connection limits, management defaults, servicing channels, hardware support, and user-experience components differ. In security interviews, avoid claiming that Server is automatically secure. A server often has a larger impact if compromised, so it requires stricter hardening, role separation, patching, monitoring, backup, and administrative controls.

## 3. What is the Windows NT architecture?

**Answer:** Windows NT is a layered operating-system architecture with user-mode and kernel-mode components. User mode contains applications, service processes, environment subsystems, and system-support processes. Kernel mode contains the executive, kernel, device drivers, hardware abstraction layer, and low-level security and memory-management functions. The executive implements services such as process management, virtual memory, I/O, object management, security reference monitoring, and interprocess communication. The separation is a security boundary because user-mode code cannot directly access kernel memory or privileged CPU instructions. However, a vulnerable or malicious kernel driver can bypass many user-mode protections, which is why driver signing, the vulnerable-driver blocklist, memory integrity, and application control matter.

## 4. What is the difference between user mode and kernel mode?

**Answer:** User mode is the restricted execution environment used by normal applications and many services. A user-mode process has its own virtual address space and must request privileged operations through system calls. Kernel mode has broad access to system memory, hardware, and privileged processor instructions. The Windows kernel, executive components, and most device drivers execute there. A crash in one user-mode process usually affects only that process, while a serious kernel-mode failure can crash or compromise the entire system. From a security perspective, gaining kernel execution is highly valuable to attackers because it can enable tampering with security tools, credentials, processes, and logs. Defenders therefore treat unexpected drivers and kernel events as high-priority evidence.

## 5. What is a system call in Windows?

**Answer:** A system call is the controlled transition through which user-mode software requests a service implemented by the operating system. Examples include creating a process, opening a file, allocating memory, or changing a registry value. Application code normally calls documented Win32 APIs; those APIs may eventually invoke lower-level Native API functions and transition into kernel mode. A system call is not the same as an ordinary function call because it crosses a privilege boundary. Security products often observe API use, system-call behavior, or resulting kernel activity, but no single layer provides perfect visibility. Attackers may call Native APIs directly or use indirect system-call techniques, so defenders should correlate process, image-load, memory, file, registry, and network telemetry.

## 6. What is the Hardware Abstraction Layer?

**Answer:** The Hardware Abstraction Layer, or HAL, hides many platform-specific hardware details from the Windows kernel and executive. It provides a consistent interface for interrupt controllers, timers, multiprocessor coordination, and other low-level hardware operations. This abstraction helps Windows run across supported hardware without requiring core operating-system components to understand every platform implementation. The HAL is not a general device-driver replacement; vendors still provide drivers for specific devices. In an interview, explain that the HAL supports portability and stability at a very low level. From a forensic perspective, normal analysts rarely examine the HAL directly, but boot-chain compromise, malicious firmware, and kernel-level tampering may affect layers beneath ordinary endpoint telemetry.

## 7. What are the major Windows boot phases?

**Answer:** On a modern UEFI system, firmware initializes hardware, verifies configured Secure Boot trust, and starts Windows Boot Manager. Boot Manager reads the Boot Configuration Data store and launches the Windows loader. The loader maps the kernel, HAL, boot-start drivers, and required registry data, then transfers control to the kernel. The kernel initializes executive subsystems and drivers, Session Manager starts user-mode system initialization, and the Service Control Manager starts services. Winlogon and related components enable interactive sign-in. Exact details vary by version and configuration. For troubleshooting or forensics, identify the failing layer: firmware, boot configuration, loader, kernel or driver initialization, services, authentication, or the user shell.

## 8. What is UEFI, and how does it differ from legacy BIOS?

**Answer:** UEFI is modern firmware that initializes hardware and starts the operating-system boot process using defined firmware interfaces and boot entries. It supports features such as GPT disks, Secure Boot, richer pre-boot services, and firmware variables. Legacy BIOS uses an older boot model that typically loads code from the disk's master boot record and has limitations associated with MBR partitioning. UEFI itself does not guarantee security; firmware must be updated, configured correctly, and protected against unauthorized changes. During incident response, unexpected UEFI settings, unauthorized boot entries, disabled Secure Boot, or firmware changes may be important, but investigators must use vendor-approved methods because careless firmware interaction can alter evidence or render a device unbootable.

## 9. What is Secure Boot?

**Answer:** Secure Boot is a UEFI feature that allows firmware to verify signatures in the pre-operating-system boot chain against trusted databases and to reject untrusted boot components according to policy. Its purpose is to reduce the risk of bootkits and unauthorized loaders executing before Windows security controls initialize. Secure Boot is one component of trusted boot; it does not scan every application or prove that a running system is uncompromised. Its effectiveness depends on protected firmware settings, sound key management, current revocation data, and a trustworthy boot chain. In forensics, Secure Boot state is useful context, but an enabled status alone cannot exclude firmware, driver, credential, or user-mode compromise.

## 10. What is Trusted Boot?

**Answer:** Trusted Boot is the Windows continuation of boot-chain verification after UEFI Secure Boot hands control to Windows. Windows validates the integrity and signatures of critical startup components, including the kernel and boot-start drivers, before they execute. Measured Boot can additionally record boot measurements in the TPM for later attestation. The important interview distinction is that Secure Boot is enforced by firmware, Trusted Boot is enforced by Windows during startup, and Measured Boot records measurements for evaluation. These controls reduce early-boot tampering but do not replace patching, endpoint detection, application control, or administrative security. A legitimately signed but vulnerable component can still create risk.

## 11. What is a TPM?

**Answer:** A Trusted Platform Module is a hardware or firmware-backed security component that can protect cryptographic keys, record platform measurements, perform attestation-related operations, and support features such as BitLocker and Windows Hello. A TPM can seal a key so it is released only when expected platform measurements and authorization conditions are satisfied. It is not a general-purpose vault that makes secrets impossible to steal, and it does not replace strong authentication or recovery planning. Administrators must understand ownership, firmware updates, clearing operations, and recovery-key storage. Investigators should document TPM and BitLocker state before changing firmware settings because such changes can trigger recovery or affect access to encrypted evidence.

## 12. What is Measured Boot?

**Answer:** Measured Boot records cryptographic measurements of firmware and boot components into TPM Platform Configuration Registers as the system starts. Those measurements can be evaluated locally or by a remote attestation service to determine whether the boot state matches policy. Unlike Secure Boot, which can prevent an untrusted component from loading, Measured Boot primarily creates evidence about what loaded. The two controls complement each other. A measurement mismatch does not automatically prove malware; firmware updates, configuration changes, or legitimate boot changes can also alter measurements. A defensible investigation compares measurements with known-good baselines and change records rather than treating any difference as malicious.

## 13. What is virtualization-based security?

**Answer:** Virtualization-based security, or VBS, uses the Windows hypervisor to create an isolated execution environment that is separated from the normal Windows kernel. Security features can place sensitive code or data in this protected environment so that even a kernel compromise has a harder time reaching it. Credential Guard and hypervisor-protected code integrity are prominent examples. VBS depends on compatible hardware, firmware, virtualization support, and policy. It can have compatibility or performance implications, so organizations should test deployments. VBS raises the cost of attacks but is not absolute isolation; vulnerabilities in the hypervisor, trusted components, firmware, or configuration can still matter.

## 14. What is Hypervisor-Protected Code Integrity?

**Answer:** Hypervisor-Protected Code Integrity, commonly surfaced as Memory Integrity, uses VBS to protect code-integrity decisions and restrict untrusted kernel-mode code. It helps enforce that kernel drivers and code meet signing and integrity policy before execution. This reduces the value of malicious or tampered drivers and makes some kernel attacks more difficult. It can expose incompatible legacy drivers, which is why staged testing and driver inventory are important. In an interview, state that HVCI complements, rather than replaces, driver signing, the vulnerable-driver blocklist, patching, and application control. An allowed signed driver may still contain an exploitable vulnerability.

## 15. What is a secured-core PC?

**Answer:** A secured-core PC is a class of device designed to combine modern hardware, firmware, and Windows security features such as TPM 2.0, Secure Boot, VBS, HVCI, and stronger protections against firmware and kernel attacks. The goal is to establish a stronger root of trust and enable important controls by design rather than relying only on post-deployment configuration. It is most valuable for high-risk users and sensitive workloads, but it does not remove the need for patching, least privilege, phishing resistance, logging, and incident response. Security posture still depends on how the device is configured and managed throughout its lifecycle.

## 16. What is the Windows Object Manager?

**Answer:** The Object Manager is an executive component that provides a uniform model for many operating-system resources, including processes, threads, files, registry keys, events, mutexes, sections, and access tokens. Objects can have names, handles, reference counts, types, and security descriptors. User-mode programs normally access objects through handles rather than direct kernel addresses. This model enables centralized lifetime management and access checking. In security analysis, handles can reveal which processes have opened sensitive files, processes, tokens, or synchronization objects. However, a handle snapshot is time-specific and should be correlated with process ancestry, privileges, loaded modules, and system activity.

## 17. What is a handle in Windows?

**Answer:** A handle is a process-specific reference to a kernel-managed object. A program receives a handle after successfully opening or creating an object such as a file, process, registry key, event, service, or token. The access rights associated with the handle are determined at creation or duplication and constrain what operations the process can perform. Handles are not globally meaningful numbers; the same numeric value in two processes can refer to different objects. During forensics or debugging, open handles may show active access to malware files, deleted files, named pipes, registry keys, or other processes. Absence of a handle does not prove that access never occurred.

## 18. What is a security boundary?

**Answer:** A security boundary is a separation that the platform intends to enforce against an attacker with specified capabilities. Examples include user-to-kernel separation, process isolation, virtual-machine isolation, and certain credential-isolation boundaries. Not every Windows feature is a security boundary. Microsoft has historically described UAC primarily as a convenience and defense-in-depth mechanism rather than a hard boundary between an administrator and the same administrator's elevated context. In interviews, avoid calling every prompt, desktop, or container a guaranteed barrier. Threat models must identify which principal is being isolated from which other principal and what level of compromise the design assumes.

## 19. What is a session in Windows?

**Answer:** A Windows session is a logical environment that groups processes, window stations, desktops, and user interaction. Session 0 is reserved for services on modern Windows, while interactive users operate in other sessions. This isolation helps prevent services from directly interacting with user desktops, reducing risks associated with the older interactive-service model. Remote Desktop users can have separate sessions on supported systems. In incident analysis, session identifiers help correlate processes and logons, but a session ID alone does not identify a person. Analysts should correlate it with logon IDs, accounts, source addresses, timestamps, and authentication records.

## 20. What are window stations and desktops?

**Answer:** A window station is a securable object that contains one or more desktop objects. A desktop provides a logical graphical surface containing windows, menus, and hooks. The interactive window station normally includes the user's standard desktop and special desktops such as the secure desktop used for certain credential and elevation prompts. Access controls restrict which processes can interact with these objects. This architecture helps isolate graphical interaction, but malicious code running in the same user context may still manipulate many user-interface elements. Security decisions should not rely solely on what a window looks like; trusted paths and protected credential interfaces are important.

## 21. What is the Windows API?

**Answer:** The Windows API is the documented programming interface exposed to applications for operating-system services such as files, processes, networking, graphics, services, registry access, and security. Many applications use the Win32 API, while newer frameworks may wrap those calls. The Native API is a lower-level interface used internally by Windows and exposed through components such as `ntdll.dll`; it is not identical to the documented Win32 surface. Security analysts should understand that behavior can be implemented through multiple APIs, so detections based on a single function name are brittle. Outcome-focused telemetry—process creation, memory protection changes, file writes, registry changes, and network connections—is usually more durable.

## 22. What is WOW64?

**Answer:** WOW64 is the compatibility subsystem that allows many 32-bit Windows applications to run on 64-bit Windows. It manages differences in execution mode, system calls, file-system redirection, and registry views. For example, a 32-bit process may be redirected from `System32` to `SysWOW64`, despite the names appearing counterintuitive. The registry can also expose separate 32-bit and 64-bit views for parts of `HKLM\Software`. In troubleshooting and forensics, analysts must record process architecture and understand redirection; otherwise they may inspect the wrong file or registry path. WOW64 does not emulate every old technology, and driver architecture must match the operating system.

## 23. What are environment variables in Windows?

**Answer:** Environment variables are name-value pairs supplied to a process and used to locate paths, identify profiles, and control application behavior. Common examples include `PATH`, `TEMP`, `USERPROFILE`, `ProgramData`, and `SystemRoot`. A child process normally inherits an environment block from its parent, which can then be modified. Security issues arise when writable directories appear early in `PATH`, when scripts trust unvalidated variables, or when malware uses environment-specific paths to remain portable. Forensic analysts should not assume every system uses the same drive letter or profile path; variables and registry-backed configuration help resolve the actual location.

## 24. What is the difference between a Windows feature, role, and service?

**Answer:** A feature is an optional operating-system capability, while a server role is a higher-level function that a Windows Server system is intended to provide, such as DNS or Hyper-V. A service is a long-running process or process-hosted component managed by the Service Control Manager. Installing a role can enable multiple features, services, firewall rules, and management tools. The terms therefore describe different layers. In security reviews, assess the complete attack surface created by a role rather than only checking whether one service is running. Remove unused roles and features, restrict service accounts, and monitor configuration changes.

## 25. How should you determine the exact Windows version and build?

**Answer:** Use more than the marketing name. Record the edition, release, build number, architecture, servicing state, and whether the system is client or server. Useful methods include `winver`, `systeminfo`, PowerShell queries such as `Get-ComputerInfo`, and the registry under `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion`. For incident response, capture the information from the affected system and corroborate it with asset-management or endpoint-management records. Build-level accuracy matters because security features, event fields, vulnerabilities, and artifact formats can change. Do not infer the build solely from a folder name or user statement.

---

# Processes, Services, Registry, Storage, and NTFS

## 26. What is a process in Windows?

**Answer:** A process is a container for an executing program. It includes a private virtual address space, one or more threads, an access token, open handles, loaded modules, environment variables, and accounting information. A process is not the executable file itself; the file is an image that can be mapped into one or many processes. Security analysts examine the image path, command line, parent process, token, integrity level, hashes, signer, loaded DLLs, network activity, and creation time. Parent-child relationships are useful but not conclusive because process creation can be brokered, re-parented in telemetry, or manipulated. Correlation across several data sources is stronger than trusting a single process tree.

## 27. What is a thread?

**Answer:** A thread is the unit scheduled for execution by Windows. Threads within the same process share the process's virtual address space and handles but have their own registers, stack, scheduling state, and thread-local storage. A process must have at least one thread to execute. Attackers may create remote threads, queue asynchronous procedure calls, or hijack existing thread contexts to execute code in another process. These behaviors can be legitimate in debuggers, security tools, and application frameworks, so detections require context. During memory analysis, suspicious thread start addresses, unbacked executable memory, and mismatches between loaded modules and execution locations can be valuable indicators.

## 28. What is a process access token?

**Answer:** An access token represents the security context under which a process or thread operates. It contains the user SID, group SIDs, privileges, integrity level, logon-session information, restrictions, and other security attributes. Windows compares token information with an object's security descriptor when deciding access. Primary tokens are associated with processes; impersonation tokens allow a thread to act temporarily on behalf of another security context. Attackers seek privileged tokens for impersonation or duplication, while defenders look for unusual token privileges, integrity transitions, and processes running under unexpected accounts. Possessing an administrator SID in a token does not always mean the process is elevated because UAC may provide a filtered token.

## 29. What is a Windows service?

**Answer:** A Windows service is a background component managed by the Service Control Manager, usually designed to start at boot or run without an interactive user. A service configuration specifies items such as the service name, executable path, start type, account, dependencies, and recovery behavior. Services may run in their own process or share a host process such as `svchost.exe`. From a security perspective, weak service executable permissions, unquoted paths, writable directories, unsafe DLL loading, and overprivileged service accounts can enable escalation. Investigators should compare service configuration with file metadata, signer information, creation events, registry entries, and expected baselines.

## 30. What is the Service Control Manager?

**Answer:** The Service Control Manager, implemented primarily through `services.exe`, maintains the database of installed services and coordinates service start, stop, control requests, dependencies, and status. Administrators interact with it through tools such as `services.msc`, `sc.exe`, PowerShell service cmdlets, WMI/CIM, and management platforms. The persistent configuration is represented in the registry, mainly under `HKLM\SYSTEM\CurrentControlSet\Services`. Security monitoring should detect new service installation, changes to image paths or accounts, and unexpected start-type changes. A service entry alone does not prove execution; correlate configuration with event logs, process telemetry, prefetch or memory, and file-system evidence.

## 31. What is `svchost.exe`?

**Answer:** `svchost.exe` is a generic host process used to run Windows services implemented as DLLs and, in some configurations, groups of related services. Multiple legitimate instances normally run with different command-line parameters, service groups, accounts, and privileges. Therefore, the process name alone is not suspicious or sufficient for validation. Analysts should inspect the executable path, signer, command line, hosted services, parent process, token, network activity, and loaded modules. Malware may imitate the name from another directory or inject into a real instance. On newer Windows versions, services may be separated into more individual host processes to improve isolation and reliability.

## 32. What are common critical Windows system processes?

**Answer:** Important processes include `System`, `smss.exe`, `csrss.exe`, `wininit.exe`, `services.exe`, `lsass.exe`, `winlogon.exe`, `explorer.exe`, and service-host processes. Their expected parentage, session, path, count, signer, and start time vary by component and Windows version. Interview answers should avoid rigid rules such as “there must always be exactly one instance,” because sessions and architecture can affect counts. A safer method is to compare observed properties with Microsoft documentation and a known-good system of the same build. Unexpected paths, unsigned replacements, anomalous parentage, user-writable locations, or unusual network behavior deserve investigation.

## 33. What is `lsass.exe`?

**Answer:** The Local Security Authority Subsystem Service enforces local security policy, supports authentication packages, creates access tokens, and handles sensitive authentication material. The legitimate executable is located in the Windows system directory and is a critical protected process on modern configurations. Credential-dumping attacks often target LSASS memory or abuse authentication mechanisms around it. Defenses include Credential Guard, LSA protection, least privilege, attack-surface reduction, protected administration, and endpoint detection. Investigators must be cautious when acquiring LSASS-related evidence because memory can contain sensitive credentials. A process named `lsass.exe` outside the expected path is highly suspicious, but path alone is not the only validation check.

## 34. What is `smss.exe`?

**Answer:** Session Manager Subsystem, `smss.exe`, is an early user-mode system process launched during startup. It performs tasks such as creating sessions, initializing environment subsystems, starting key processes, and handling configured session-management operations. Because it appears early in the boot chain and has important responsibilities, unexpected images or configuration related to Session Manager can be security-relevant. Analysts may review the `Session Manager` registry keys, image path, signer, parentage, and boot timeline. Exact child-process relationships vary across Windows versions, so investigators should rely on build-appropriate references rather than memorized diagrams alone.

## 35. What is `csrss.exe`?

**Answer:** Client Server Runtime Subsystem, `csrss.exe`, is a critical user-mode process involved in console handling, process and thread support, and other subsystem responsibilities. Multiple legitimate instances can exist because sessions are separated. The genuine binary is stored in the Windows system directory and is protected as a critical process. Malware may use similar names or place a fake copy elsewhere. Analysts should validate image path, signature, session ID, parentage, start time, loaded modules, and memory behavior. Terminating the legitimate process can crash the system, so live-response actions should be deliberate and authorized.

## 36. What is `winlogon.exe`?

**Answer:** `winlogon.exe` manages parts of interactive sign-in and logon-session behavior, including responding to the secure attention sequence and coordinating user-session initialization. It works with other authentication and shell components rather than independently validating every credential. A legitimate instance is associated with an interactive session and runs from the Windows system directory. Security analysts investigate unexpected children, injected code, unusual modules, credential-provider abuse, and persistence mechanisms that alter logon-related registry values. As with other system processes, normal counts and parent relationships can depend on sessions and version.

## 37. What is the Windows Registry?

**Answer:** The Windows Registry is a hierarchical database used by Windows and applications to store configuration and state. It is organized into keys, subkeys, values, and data types, exposed through logical root keys such as `HKEY_LOCAL_MACHINE` and `HKEY_CURRENT_USER`. The logical view is backed by hive files and can include volatile data. Security and forensic value comes from persistence settings, services, user activity, device history, application configuration, and system state. The Registry is not a single file, and a key's last-write time applies to the key rather than every individual value. Interpretation should account for transaction logs, control sets, 32/64-bit views, and user-specific hives.

## 38. What are Registry hives?

**Answer:** A hive is a logical group of Registry keys and values backed by one or more files loaded into memory. Common system hives include `SYSTEM`, `SOFTWARE`, `SAM`, `SECURITY`, and `DEFAULT` under the system configuration directory. User-specific data is commonly stored in `NTUSER.DAT`, while `UsrClass.dat` stores additional per-user shell and class information. Hive transaction logs can be needed to recover the most current consistent state. Investigators should acquire hives and associated logs together when possible, preserve original timestamps, and document whether the source was live, shadow-copied, or imaged.

## 39. What is `CurrentControlSet`?

**Answer:** `CurrentControlSet` is a runtime symbolic view of the control set Windows selected for the current boot. The underlying `SYSTEM` hive may contain multiple numbered control sets such as `ControlSet001` and `ControlSet002`. The `Select` key helps identify current, default, last-known-good, and failed control sets. During offline analysis, `CurrentControlSet` may not exist as a literal stored key, so tools resolve the selected numbered set. This matters when examining services, drivers, network configuration, time-zone data, and device state. Analysts should state which control set was interpreted and consider whether another control set reflects an earlier boot configuration.

## 40. What is Registry redirection and reflection?

**Answer:** On 64-bit Windows, parts of the Registry expose separate views to 32-bit and 64-bit applications. WOW64 redirection allows software of each architecture to see the appropriate view, commonly affecting areas under `HKLM\Software`. Older Windows versions used reflection for some keys to keep views synchronized, while modern behavior is more selective. Investigators and administrators must know which process architecture a tool uses because it may read a different view. PowerShell, `reg.exe`, and forensic tools can produce apparently conflicting results if architecture is ignored. Always record the path, view, and acquisition method.

## 41. What is NTFS?

**Answer:** NTFS is the primary Windows file system for many internal volumes. It supports metadata-rich file records, access-control lists, alternate data streams, compression, encryption, reparse points, sparse files, journaling, hard links, quotas, and large volumes. Its central metadata file is the Master File Table. Forensics benefits from MFT records, timestamps, file references, the USN Change Journal, `$LogFile`, and other metadata. NTFS journaling is designed for file-system consistency, not complete forensic history. Records can be reused, journals can wrap, timestamps can be altered, and SSD behavior can reduce deleted-data recovery. Findings should be corroborated.

## 42. What is the Master File Table?

**Answer:** The Master File Table, `$MFT`, contains at least one record for every file and directory on an NTFS volume, including NTFS metadata files. A record can hold attributes such as file names, timestamps, security identifiers, and either resident data or references to nonresident data runs. When a file is deleted, its record is marked available for reuse; it is not necessarily erased immediately. Forensic analysis can recover names, metadata, and sometimes content references from active or deleted records. However, reused records, multiple file-name attributes, hard links, and timestamp manipulation can complicate interpretation. The sequence number in a file reference helps detect some record reuse.

## 43. What are resident and nonresident NTFS attributes?

**Answer:** An NTFS attribute is resident when its content fits inside the MFT record; otherwise it is nonresident and the record stores data-run mappings to clusters elsewhere on the volume. Small files may therefore have content entirely inside the MFT. Large files, fragmented files, and many metadata streams are normally nonresident. This distinction matters in recovery because deleting a small resident file may leave its content in an unallocated MFT record, while nonresident content depends on whether clusters have been reused or trimmed. A forensic tool should report attribute type and allocation state rather than simply saying a file was “recovered.”

## 44. What are NTFS timestamps?

**Answer:** NTFS commonly stores timestamps in both `$STANDARD_INFORMATION` and `$FILE_NAME` attributes. These can include creation, content modification, metadata change, and access times. Their update behavior differs by operation, Windows version, application, file-system settings, copy method, archive extraction, and attacker manipulation. The familiar MACB terminology is useful but can oversimplify the actual fields. Timestamps are evidence of recorded file-system state, not direct proof that a person performed an action at that moment. Analysts should compare multiple timestamp sets, USN records, event logs, application artifacts, and external sources, while also validating time zone and clock accuracy.

## 45. What is the USN Change Journal?

**Answer:** The NTFS USN Change Journal records that changes occurred to files and directories on a volume and identifies reasons such as creation, deletion, rename, data extension, or close. It is efficient for indexing and replication because applications can query changes without scanning the whole volume. Forensics can use it to identify file activity even when a file is gone. The journal does not contain full prior file content, may combine reason flags, can wrap and discard old records, and can be deleted or recreated. A missing record does not prove no change occurred. Analysts should correlate journal entries with MFT records and `$LogFile`.

## 46. What is NTFS `$LogFile`?

**Answer:** `$LogFile` is the NTFS transaction log used to help restore file-system metadata consistency after a crash. It records redo and undo information for metadata operations, not a complete user-activity audit trail. Forensic tools can sometimes recover recent evidence of file creation, deletion, rename, or metadata changes, especially when combined with the MFT and USN Journal. Its content is circular and version-dependent, and interpretation is technically complex. An entry indicates a file-system transaction, not necessarily the intent or identity of a user. Analysts should use validated parsers and preserve raw metadata for independent review.

## 47. What are alternate data streams?

**Answer:** NTFS allows a file to contain multiple named data streams. The unnamed stream is what users normally treat as the file's content, while additional streams can store metadata or arbitrary data. Windows uses streams legitimately, for example the `Zone.Identifier` stream that can record download-origin information. Attackers can hide payloads or configuration in streams, but alternate streams are not inherently malicious and are not invisible to proper tools. Copying to a file system that does not support streams can remove them. Forensic collection should preserve NTFS semantics and explicitly enumerate streams rather than relying on normal directory listings.

## 48. What are reparse points?

**Answer:** A reparse point is NTFS metadata that allows a file-system filter or Windows component to apply special handling to a file or directory. Symbolic links, junctions, mount points, cloud placeholders, and certain application features use reparse points. They can redirect access to another path or volume, which creates security and forensic pitfalls. A recursive collection can unintentionally leave the intended directory tree, loop, or acquire remote content. Attackers may abuse reparse behavior in privilege-escalation or deletion attacks. Tools should identify the reparse tag, avoid blindly following links, and record both the link object and target context.

## 49. What is Volume Shadow Copy Service?

**Answer:** Volume Shadow Copy Service coordinates point-in-time snapshots through providers, writers, and requesters so applications and backup tools can capture consistent data. Shadow copies may preserve older versions of files, Registry hives, and other artifacts that have changed or been deleted from the live volume. They are highly valuable in ransomware and timeline investigations. They are not guaranteed backups: retention is limited, snapshots can be deleted, application consistency varies, and accessing them can alter the live system if done carelessly. Investigators should enumerate snapshot identifiers and times, acquire relevant data read-only where feasible, and compare snapshots with the active file system.

## 50. What happens when a file is deleted in Windows?

**Answer:** The outcome depends on the application, file system, storage device, and deletion method. A normal shell deletion may move the item into the Recycle Bin, creating metadata that records the original path and deletion time. A permanent deletion usually marks directory and MFT structures as available and marks clusters unallocated; content can remain until overwritten. On SSDs, TRIM and garbage collection may make recovery difficult or impossible. Cloud synchronization, backup, shadow copies, application caches, MFT records, and journals may retain evidence. Therefore, “deleted” does not mean securely erased, but neither does it guarantee recoverable content.

---

# Identity, Authentication, Authorization, and Permissions

## 51. What is the difference between identification, authentication, and authorization?

**Answer:** Identification is the claim of an identity, such as a username or certificate subject. Authentication verifies that claim using evidence such as a password, smart card, FIDO2 key, biometric gesture, or cryptographic proof. Authorization determines what the authenticated security principal is allowed to do. Windows then performs access checks using tokens, privileges, groups, integrity levels, and object security descriptors. These stages should not be confused: a successful logon proves that an authentication mechanism accepted evidence for an account, not that a particular human was physically present, and authorization can still deny access after authentication succeeds.

## 52. What is a security principal?

**Answer:** A security principal is an entity that can be authenticated or assigned permissions, such as a user, computer, service account, managed identity, or security group. Windows identifies principals with security identifiers rather than relying only on display names. A renamed account usually retains the same SID, while a deleted and recreated account with the same name receives a different SID. This distinction matters in permissions and forensics because an ACL can contain an unresolved SID after the original principal is gone. Analysts should resolve SIDs using trustworthy domain or local account data and avoid treating an account name alone as a stable identity.

## 53. What is a SID?

**Answer:** A Security Identifier is a variable-length value that uniquely identifies a Windows security principal within the issuing authority's scope. Tokens and ACLs use SIDs for access decisions. Well-known SIDs represent built-in identities and groups, while local and domain principals include an authority portion and a relative identifier. The RID is not meaningful without the rest of the SID. When an account is deleted, its SID is not reused for a new principal, which is why permissions can show orphaned entries. In investigations, SIDs provide more durable correlation than names, but they still identify accounts or groups, not necessarily the human who used them.

## 54. What is the Security Accounts Manager?

**Answer:** The Security Accounts Manager, or SAM, stores local account information on a Windows system. Its data is represented in the protected `SAM` Registry hive and is used with other security components during local authentication. On a domain controller, domain account data is held in Active Directory rather than the local SAM model used by member systems. Attackers may target SAM-related material for offline credential attacks, so acquisition and access should be tightly controlled. Credential hashes are not plaintext passwords, and possession of a hash can still be dangerous because it may support cracking or pass-the-hash techniques depending on the environment.

## 55. What is the difference between a local account, domain account, Microsoft account, and Entra ID account?

**Answer:** A local account is managed by one Windows device. A domain account is managed by Active Directory Domain Services and can authenticate across domain resources. A Microsoft account is a consumer cloud identity used with Microsoft services and Windows features. A Microsoft Entra ID account is an organizational cloud identity used for work or school access and device management. The sign-in user experience can look similar while credential storage, authentication protocols, policy, recovery, and logging differ. Analysts should determine the actual identity provider and device join state before interpreting logons or troubleshooting access.

## 56. What is an access token?

**Answer:** An access token is the kernel-managed representation of a logon security context. It includes the user SID, group memberships, privileges, integrity level, token type, restrictions, and other claims or attributes. A process normally has a primary token, while a thread can use an impersonation token. Access checks compare token information to an object's security descriptor and mandatory integrity policy. Tokens are created after authentication but can be filtered, restricted, duplicated, or impersonated. Investigators should inspect token properties rather than inferring privilege solely from the account name.

## 57. What are Windows privileges?

**Answer:** Privileges are token rights that authorize system-wide operations not expressed as ordinary object permissions. Examples include debugging processes, backing up files while bypassing normal read checks, restoring files, loading drivers, changing system time, or impersonating clients. A privilege can be present but disabled until a process enables it. Some privileges are extremely powerful and can lead to full compromise. Least-privilege reviews should examine actual user-right assignments and service accounts, not only membership in Administrators. A security event showing a privileged logon is context for investigation, not proof that every privilege was exercised.

## 58. What is the difference between a right, privilege, and permission?

**Answer:** In Windows terminology, user rights include logon rights and privileges assigned through security policy. A privilege allows a system-level operation such as backup or debug. A permission is an access right on a securable object, such as read data on a file or start access on a service. Administrators often use the terms loosely, but precise language improves troubleshooting. A user may have permission to modify a file without holding a special privilege, or may use `SeBackupPrivilege` to read files despite ordinary DACL denial when a backup-aware application requests backup semantics.

## 59. What is a security descriptor?

**Answer:** A security descriptor is the data structure that defines ownership, discretionary access control, auditing, and mandatory integrity information for a securable object. It can contain an owner SID, group SID, DACL, SACL, and control flags. Security descriptors apply to files, Registry keys, processes, services, named pipes, and many other objects. A null DACL, an empty DACL, and a missing DACL have different meanings and should not be confused. Investigators and administrators should preserve the original descriptor when collecting evidence because copying content alone can lose permissions, inheritance, ownership, and audit configuration.

## 60. What is a DACL?

**Answer:** A Discretionary Access Control List contains access control entries that allow or deny specified rights to SIDs. Windows evaluates relevant entries during access checks, considering the requested access, token groups, deny and allow entries, inheritance, and object type. Explicit deny entries can override matching allows, but effective access can become complex because of group memberships and inherited entries. A null DACL generally grants broad access, while an empty DACL grants no access. In security reviews, writable paths used by privileged services or scheduled tasks are especially important because they can create escalation opportunities.

## 61. What is a SACL?

**Answer:** A System Access Control List specifies which access attempts should generate audit events and can also contain mandatory integrity labels. Auditing only occurs if the relevant system audit policy is enabled; configuring a SACL alone may produce no events. Excessive auditing can overwhelm logs, while insufficient auditing leaves gaps. A good design targets high-value objects, sensitive changes, and important failure conditions, then forwards and protects the resulting logs. SACL evidence indicates configured audit intent, not guaranteed historical completeness, because logs may roll over, collection may fail, or policy may have changed.

## 62. What is ACL inheritance?

**Answer:** Inheritance allows child objects to receive access control entries from parent containers. Entries can be marked to apply to files, subfolders, or both, and a child can preserve or disable inherited entries. Inheritance simplifies administration but can create broad unintended access when a high-level folder grants excessive permissions. Moving or copying an object can also affect inheritance differently depending on volume and operation. Troubleshooting should inspect explicit versus inherited entries and effective access. Forensic reporting should record the descriptor at the relevant time because later inheritance changes can alter current permissions without proving what permissions existed earlier.

## 63. What is ownership in Windows security?

**Answer:** The owner of a securable object has special authority to change its DACL, even if ordinary permissions would otherwise deny access. Ownership is therefore security-relevant and should not be treated as descriptive metadata only. Administrators and certain privileged principals can take ownership, after which they can grant themselves access. Changing ownership can be legitimate during recovery or migration, but unexpected ownership changes on system files, services, Registry keys, or sensitive data warrant investigation. Audit policy and object SACLs may be needed to capture those changes prospectively.

## 64. What is mandatory integrity control?

**Answer:** Mandatory Integrity Control assigns integrity levels to tokens and objects, such as low, medium, high, and system. Before the normal DACL check, Windows applies a mandatory policy that can block lower-integrity subjects from writing to higher-integrity objects. Standard desktop applications commonly run at medium integrity, elevated administrative processes at high, and certain sandboxed content at low or app-container levels. Integrity levels are defense in depth, not a replacement for identity permissions. A medium-integrity process can still damage data the user is authorized to modify, and an attacker who obtains elevation can cross the boundary.

## 65. What is User Account Control?

**Answer:** User Account Control reduces routine use of full administrative privileges. An administrator normally receives a filtered token for standard work and can request elevation, while a standard user may be prompted for administrative credentials. Elevation consent occurs on a protected desktop depending on policy. UAC helps limit accidental changes and some malware behavior, but it is not a complete security boundary against an attacker already executing as the same administrator. Organizations should combine UAC with standard-user operation, application control, credential protection, patching, and privileged-access management.

## 66. What are split tokens?

**Answer:** When an administrator signs in with UAC enabled, Windows can create both a full administrative token and a filtered token. The user shell normally runs with the filtered token; approved elevation launches a process with the full token. This is called a split-token model. It explains why membership in Administrators does not mean every process has high integrity or all administrative SIDs enabled. Analysts should examine token elevation type and integrity level for the specific process. Disabling UAC changes this model and usually weakens security and compatibility assumptions.

## 67. What is the secure attention sequence?

**Answer:** The secure attention sequence, commonly `Ctrl+Alt+Delete`, is handled by Windows in a way ordinary user-mode applications cannot normally intercept. It provides a trusted path to security functions such as sign-in, password change, lock, and Task Manager. Its value is that malware cannot simply draw an identical window and receive the key sequence through normal input hooks. It is not a guarantee that the whole system is uncompromised; kernel or credential-provider compromise can still matter. Organizations may require the sequence before logon as a phishing-resistance measure on managed devices.

## 68. How does Windows password authentication work at a high level?

**Answer:** Windows does not normally store a user's reusable plaintext password for comparison. Local and domain authentication use protocol-specific cryptographic values and challenge-response or ticket mechanisms. Local accounts are associated with SAM-held password-derived data, while domain authentication is typically handled by domain controllers using Kerberos or, where necessary, NTLM. During interactive sign-in, Windows authentication packages validate credentials and LSASS creates a logon session and token. Exact behavior depends on account type, credential provider, cached logon state, network availability, and policy. Password hashes remain sensitive because they can support offline cracking or authentication abuse.

## 69. What is Kerberos?

**Answer:** Kerberos is a ticket-based network authentication protocol used primarily in Active Directory domains. A client obtains a Ticket Granting Ticket from the Key Distribution Center and later requests service tickets for specific services. This supports mutual authentication and avoids sending the password to every service. Kerberos depends heavily on time synchronization, service principal names, DNS, account keys, and domain-controller availability. Common security risks include stolen tickets, delegation abuse, weak service-account passwords, and excessive ticket lifetimes. A ticket shows authentication for a principal and service context, but it does not by itself prove which human initiated the activity.

## 70. What is NTLM?

**Answer:** NTLM is a challenge-response authentication family retained for compatibility when Kerberos or newer methods cannot be used. It relies on password-derived secret material and does not provide the same ticket-based design or mutual-authentication properties as Kerberos. NTLM is vulnerable to relay in improperly protected protocols and to pass-the-hash abuse when credential material is stolen. Organizations should reduce and monitor NTLM use, enable protections such as SMB signing where appropriate, and address applications that prevent migration. Analysts should inspect authentication package, source, destination, account, and protocol context rather than assuming every NTLM event is malicious.

## 71. What is pass-the-hash?

**Answer:** Pass-the-hash is an attack in which an adversary uses a stolen NTLM password hash or equivalent credential material to authenticate without knowing the plaintext password. It exploits the fact that some authentication flows treat possession of the hash-derived secret as sufficient. Defenses include Credential Guard, protecting administrative credentials, restricting lateral movement, using local administrator password management, reducing NTLM, tiering administration, and detecting unusual logons. Changing the password invalidates the old hash, but incident response must also identify how the hash was obtained and where it was used.

## 72. What is pass-the-ticket?

**Answer:** Pass-the-ticket uses stolen Kerberos tickets to access services as the ticket's principal. A service ticket may provide access to one service, while a stolen TGT can be used to request additional service tickets until it expires or is invalidated. Defenses include Credential Guard, privileged-account hygiene, short exposure of admin sessions, endpoint protection, and detection of anomalous ticket use. Resetting a user password may not immediately invalidate every existing ticket in all cases, so responders should follow domain-specific containment procedures. Ticket evidence must be correlated with endpoint, domain-controller, and service logs.

## 73. What is Credential Guard?

**Answer:** Credential Guard uses VBS to isolate selected credential material from the normal operating system, protecting NTLM hashes, Kerberos TGTs, and some domain credentials from many memory-scraping attacks. Even malware with administrative rights in the normal kernel has a harder time extracting isolated secrets. It does not protect every credential type, local SAM data, or credentials entered into unsafe applications, and it does not stop an attacker from abusing a currently authenticated session. Compatibility requirements must be assessed, especially for legacy authentication and delegation. Microsoft enables it by default on some modern eligible domain-joined systems, but administrators should verify actual state.

## 74. What is LSA protection?

**Answer:** LSA protection configures LSASS as a protected process light so that only appropriately signed and trusted code can load into or access it using sensitive rights. This helps resist credential theft and unauthorized authentication-package injection. It is separate from Credential Guard, although the controls complement each other. Compatibility issues can occur with legacy security or authentication plug-ins, so audit and staged deployment are useful. Investigators should check configured and running state, related event logs, and any blocked plug-ins. A process-protection setting does not make credentials invulnerable to phishing, token theft, or compromised endpoints.

## 75. What is Windows Hello for Business?

**Answer:** Windows Hello for Business replaces reusable password sign-in with asymmetric key-based authentication protected by the device, often using the TPM. The user unlocks the key with a gesture such as a PIN or biometric. The PIN is device-bound and is not simply a shorter domain password transmitted to servers. Depending on deployment, certificate trust, key trust, or cloud Kerberos trust can be used. This reduces phishing and password-replay risk, but device enrollment, recovery, attestation, and identity governance remain important. Biometric data and PINs should be understood as local unlock factors for protected credentials, not direct network passwords.

---

# Windows Networking, Remote Access, and Management

## 76. What is the Windows networking stack?

**Answer:** The Windows networking stack is the set of kernel and user-mode components that implement network interfaces, TCP/IP, sockets, filtering, name resolution, authentication, and application protocols. Applications commonly use Winsock, while the kernel handles transport, IP routing, interface drivers, and packet processing. Windows Filtering Platform allows firewalls and security products to inspect or filter traffic at defined layers. Troubleshooting should move from physical link and interface configuration to IP, routing, DNS, transport, authentication, firewall, and application behavior. A successful ping does not prove that a service, port, certificate, or application is working.

## 77. What is Winsock?

**Answer:** Windows Sockets, or Winsock, is the programming interface used by Windows applications for network communication. It provides socket operations for protocols such as TCP and UDP and maps application requests to the underlying networking stack. The Winsock catalog can include layered or namespace providers, and corruption or malicious modification can disrupt networking. Commands such as `netsh winsock show catalog` and, in controlled troubleshooting, `netsh winsock reset` may help, but resetting can affect installed software and should not be the first step. Forensics should preserve relevant configuration before destructive repair.

## 78. What is an IP address, subnet mask, default gateway, and DNS server?

**Answer:** An IP address identifies an interface in an IP network. The subnet mask or prefix length determines which addresses are considered directly reachable on the local subnet. The default gateway is the router used when no more-specific route exists. DNS servers resolve names to addresses and support service discovery. Windows can receive these settings through DHCP or static configuration. Troubleshooting should use `ipconfig /all`, route tables, DNS queries, and interface state rather than relying only on the graphical settings page. A correct IP address does not guarantee correct DNS, routing, firewall, proxy, or application configuration.

## 79. How does DHCP work in Windows?

**Answer:** DHCP dynamically supplies network configuration such as an IP address, prefix, default gateway, DNS servers, and lease duration. A typical IPv4 exchange uses Discover, Offer, Request, and Acknowledge messages, though renewal behavior differs after a lease is established. Windows records client configuration and can use automatic private addressing if no DHCP server responds. Enterprise DHCP can integrate with DNS and authorization controls. Rogue DHCP servers can redirect traffic or cause outages, so switch protections and monitoring are valuable. The command `ipconfig /release` and `/renew` can test lease behavior, but analysts should capture existing configuration first.

## 80. What is APIPA?

**Answer:** Automatic Private IP Addressing allows an IPv4 interface to self-assign an address in `169.254.0.0/16` when DHCP is configured but unavailable. It enables limited local-link communication but usually provides no default gateway or enterprise DNS. An APIPA address is therefore commonly a symptom of DHCP reachability, VLAN, wireless, switch, driver, or service problems. It is not itself proof that the network adapter is defective. Troubleshooting should check link state, DHCP client service, packet exchange, VLAN assignment, relay configuration, and server scope availability.

## 81. How does Windows DNS resolution work?

**Answer:** Windows name resolution can consult the local cache, hosts file, configured DNS servers, and protocol-specific mechanisms depending on the name and policy. Domain environments rely heavily on DNS SRV records for locating services such as domain controllers. Applications may also implement their own DNS behavior or encrypted DNS. Useful tools include `Resolve-DnsName`, `nslookup`, `ipconfig /displaydns`, and packet capture. A cached answer may differ from current authoritative data, and flushing the cache can destroy useful incident evidence. During response, collect the cache and relevant logs before clearing it.

## 82. What is the hosts file?

**Answer:** The hosts file is a local static mapping of hostnames to IP addresses, typically located under the Windows system directory. Entries can override normal DNS resolution for matching names. It is useful for testing and controlled overrides but can be abused to redirect users to malicious systems or block security services. Analysts should inspect content, file metadata, permissions, signer or process activity that changed it, and endpoint telemetry. A suspicious entry is evidence of redirection configuration, not proof that a connection succeeded or that the user saw the destination.

## 83. What is NetBIOS over TCP/IP?

**Answer:** NetBIOS over TCP/IP supports legacy name and session services used by older Windows networking. It can use ports 137, 138, and 139 and may participate in local name resolution when modern DNS-based methods are unavailable. Legacy broadcast and name-resolution behavior can expose information and enable spoofing or poisoning attacks. Modern environments should reduce unnecessary dependence on NetBIOS and validate application compatibility before disabling it. Analysts may encounter NetBIOS names in logs, packet captures, and authentication events, but should map them carefully to current DNS names and assets.

## 84. What is LLMNR?

**Answer:** Link-Local Multicast Name Resolution is a local-network fallback mechanism for resolving names when DNS does not provide an answer. It uses multicast on the local link. Attackers can respond fraudulently and induce clients to authenticate to them, potentially capturing or relaying NTLM exchanges. Organizations commonly disable LLMNR where it is not required, improve DNS reliability, reduce NTLM, and deploy protections against relay. A captured LLMNR query shows that a name-resolution attempt occurred; it does not by itself prove credentials were captured or a connection completed.

## 85. What is SMB?

**Answer:** Server Message Block is the Windows protocol family used for file, printer, named-pipe, and other network services. Modern Windows primarily uses SMB 2 and SMB 3 over TCP port 445. Features can include signing, encryption, multichannel, and transparent failover depending on version and configuration. SMB 1 is obsolete and should be removed unless a documented legacy requirement exists. Security reviews should assess authentication, share permissions, NTFS permissions, signing, encryption, guest access, lateral-movement exposure, and patching. Effective file access is constrained by both share and NTFS controls, with the most restrictive effective combination applying over the network.

## 86. What is SMB signing?

**Answer:** SMB signing adds integrity protection and peer authentication to SMB messages using session keys, helping detect tampering and reducing certain man-in-the-middle and relay attacks. It does not encrypt file content, so SMB encryption or another secure channel is needed for confidentiality. Whether signing is required, supported, or negotiated depends on client, server, version, and policy. Enabling mandatory signing can have compatibility or performance implications in some environments, but modern hardware typically reduces the cost. Analysts should verify the negotiated session properties rather than assuming policy alone reflects actual traffic.

## 87. What is SMB encryption?

**Answer:** SMB encryption protects SMB data in transit from eavesdropping and tampering without requiring IPsec. It is available in modern SMB versions and can be configured per share or server, depending on platform. Encryption does not correct weak permissions, compromised endpoints, or stolen credentials. It also reduces visibility for network sensors that cannot inspect endpoint-encrypted traffic, increasing the importance of host telemetry. Administrators should confirm protocol version and negotiation and should not confuse SMB encryption with BitLocker, which protects data at rest on a volume.

## 88. What is a Windows file share?

**Answer:** A file share exposes a local folder over SMB using a share name and share-level permissions. Access to files is also evaluated against NTFS permissions, so administrators must consider both layers. Hidden administrative shares such as `C$` and `ADMIN$` support remote administration and are restricted to appropriate administrators. Security problems often arise from broad groups, inherited NTFS rights, guest access, stale shares, or sensitive data placed in permissive folders. Auditing should include share creation and access to high-value content. A share listing does not prove that a user successfully opened a file.

## 89. What is Windows Defender Firewall?

**Answer:** Windows Defender Firewall is a host-based stateful firewall integrated with Windows. It applies inbound and outbound rules by profile, interface, program, service, protocol, port, address, and security condition. The Domain, Private, and Public profiles allow policy appropriate to network trust context. Disabling the firewall for troubleshooting is risky and can invalidate test results; create narrow temporary rules and log the change instead. Central management can use Group Policy, Intune, PowerShell, or security tooling. Firewall logs and WFP events provide useful evidence but may be incomplete if logging was not configured before an incident.

## 90. What are Windows network profiles?

**Answer:** Windows categorizes network connections into Domain, Private, or Public profiles, which affect firewall and discovery behavior. A domain profile is selected when Windows can authenticate the network to the joined domain under expected conditions; it is not merely a user-selected label. Private is intended for trusted non-domain networks, and Public applies stricter defaults for untrusted networks. Misclassification can block required services or expose unnecessary ones. Troubleshooting should determine why the profile was selected and review Network Location Awareness, DNS, domain connectivity, and policy rather than simply forcing a profile.

## 91. What is Windows Filtering Platform?

**Answer:** Windows Filtering Platform is a set of APIs and system services that allow Windows Firewall and security products to filter, inspect, modify, or authorize network traffic at multiple points in the stack. It provides callout and filter layers for applications, transport, network, and other paths. WFP enables more precise control than a simple port filter, but complex third-party filters can also cause performance or connectivity problems. Investigators may use WFP audit events, firewall logs, and filter configuration to understand blocking or allowed traffic. Configuration state should be captured before resetting the firewall.

## 92. What is Remote Desktop Protocol?

**Answer:** Remote Desktop Protocol provides graphical remote sessions to Windows systems, commonly over TCP and UDP port 3389, though gateways and custom configurations can change the path. Security depends on Network Level Authentication, strong authentication, patching, restricted exposure, MFA through an appropriate access layer, certificate validation, and least privilege. Direct Internet exposure is high risk. Logs from the Security channel, Terminal Services channels, firewalls, gateways, and identity systems should be correlated. A successful RDP logon event identifies an account and session context but does not automatically identify the human operator.

## 93. What is Network Level Authentication?

**Answer:** Network Level Authentication requires a user to authenticate before a full Remote Desktop session and interactive desktop are created. It reduces resource consumption and exposure of the logon interface to unauthenticated clients. NLA generally uses CredSSP and underlying Windows authentication. It improves security but does not make an Internet-exposed RDP service safe by itself. Stolen credentials, weak passwords, unpatched systems, or compromised clients remain risks. Organizations should place RDP behind VPN, Remote Desktop Gateway, Zero Trust access, or other controlled paths and monitor authentication.

## 94. What is WinRM?

**Answer:** Windows Remote Management is Microsoft's implementation of the WS-Management standard and is used by PowerShell remoting and administrative tools. It commonly listens on HTTP 5985 or HTTPS 5986, but the security of HTTP mode depends on the authentication and message-protection configuration rather than the absence of TLS alone. Kerberos provides strong protection in domain scenarios; HTTPS is useful across trust boundaries. Administrators should restrict listeners, allowed hosts, authentication methods, firewall scope, and endpoint permissions. Logs from WinRM and PowerShell can show remote activity if enabled and retained.

## 95. What is PowerShell remoting?

**Answer:** PowerShell remoting executes commands in remote PowerShell sessions, commonly using WinRM. It supports one-to-one and one-to-many administration and can use constrained endpoints or Just Enough Administration to limit capability. It is legitimate and powerful, which makes it attractive to attackers who obtain credentials. Disabling it everywhere may harm administration without eliminating alternatives. Better controls include privileged-access workstations, role-constrained endpoints, strong authentication, network restriction, script-block logging, module logging, transcription where appropriate, and centralized event collection.

## 96. What is WMI?

**Answer:** Windows Management Instrumentation provides a standardized management model and query interface for system information and operations. Administrators use it locally and remotely through tools, scripts, and management products. Attackers may abuse WMI for remote execution, discovery, persistence through permanent event subscriptions, or process creation. Defenders should monitor WMI activity, repository changes, consumer-filter bindings, unusual parent processes, and remote authentication. WMI use is common, so detections should consider account, source host, namespace, class, command, and timing rather than treating every query as malicious.

## 97. What is RPC?

**Answer:** Remote Procedure Call is an interprocess and network communication framework used by many Windows services. RPC can use the Endpoint Mapper on TCP 135 and dynamically assigned ports for specific services. Therefore, opening only port 135 rarely enables a complete RPC application. Firewalls should use service-aware rules and constrained dynamic ranges where required. RPC exposure can expand attack surface, so only necessary services should be reachable. Troubleshooting involves endpoint registration, authentication, name resolution, firewall policy, and service state. An RPC error is a transport or service symptom, not a diagnosis by itself.

## 98. What is a VPN connection in Windows?

**Answer:** A virtual private network creates an authenticated and usually encrypted tunnel between a Windows client and a VPN gateway. Windows supports multiple protocols and can be managed through profiles, MDM, or third-party clients. Security depends on protocol choice, certificate or credential strength, gateway configuration, split-tunneling policy, DNS behavior, routing, posture checks, and MFA. A connected status does not prove all traffic uses the tunnel. Troubleshooting should inspect routes, interface metrics, DNS suffixes, proxy settings, certificate chains, and gateway logs.

## 99. What commands are useful for Windows network troubleshooting?

**Answer:** Common tools include `ipconfig`, `Get-NetIPConfiguration`, `Get-NetAdapter`, `Get-NetRoute`, `route print`, `Resolve-DnsName`, `nslookup`, `Test-NetConnection`, `ping`, `tracert`, `pathping`, `arp`, `Get-NetTCPConnection`, `netstat`, `netsh`, and packet capture tools. The correct sequence matters: establish interface and address state, then routing, DNS, port reachability, TLS or authentication, and application behavior. Running many commands without a hypothesis creates noise. During an incident, preserve outputs with timestamps and avoid commands that clear caches or reset stacks until volatile evidence is captured.

## 100. How would you troubleshoot a Windows host that can reach IP addresses but not websites by name?

**Answer:** First confirm the symptom with `Resolve-DnsName` or `nslookup` rather than only a browser. Review `ipconfig /all` for DNS servers and suffixes, test reachability to those servers, and query a known name directly against the configured resolver. Inspect the DNS client cache, hosts file, VPN and proxy settings, firewall rules, and whether the problem affects one application or the whole system. Check time if DNS security or domain services are involved. Capture relevant evidence before flushing caches. If direct DNS queries succeed but the browser fails, investigate proxy, TLS, endpoint security, and application configuration.

---

