# 3.-Windows-Log-Analysis
# Windows Log Analysis & Threat Detection

## Overview

This project demonstrates hands-on Windows security log analysis and threat detection using Windows Event Logs and Sysmon telemetry.

The investigation focused on identifying suspicious authentication activity, detecting successful remote access, analyzing malicious process execution, identifying network IOCs, and investigating suspicious user-account activity.

This practical investigation was completed as part of my **SOC Analyst L1 hands-on training** using **TryHackMe's Windows Threat Detection 1** room.

---

## Investigation Objectives

- Analyze Windows Security Event Logs
- Investigate failed authentication attempts
- Detect potential RDP brute-force activity
- Identify successful RDP logons
- Analyze suspicious process execution using Sysmon
- Investigate suspicious DNS activity
- Extract potential network IOCs
- Detect suspicious user-account creation
- Investigate privileged group membership changes
- Correlate multiple events to reconstruct attacker activity
- Document findings from a SOC Analyst perspective

---

## Tools & Technologies

- Windows Event Viewer
- Windows Security Event Logs
- Sysmon
- TryHackMe
- Windows Event Logs (`.evtx`)
- Event ID analysis
- Process and network telemetry

---

# 1. RDP Brute-Force Detection

## Objective

Identify repeated failed authentication attempts that may indicate an RDP brute-force or password-guessing attack.

### Event Investigated

**Event ID 4625 — An account failed to log on**

### Analysis

Multiple Event ID 4625 events were observed within a short time period, indicating repeated authentication failures against the Windows system.

The account name, logon type, timestamp, and other available event details were reviewed to determine whether the activity represented suspicious authentication behavior.

### Evidence

![RDP Brute Force Detection](screenshots/rdp-bruteforce.png)

### Finding

> Multiple Event ID 4625 failed logon events were observed within a short time window. The repeated authentication failures were investigated as potential RDP brute-force or password-guessing activity.

---

# 2. Successful RDP Logon

## Objective

Determine whether the observed authentication attempts resulted in a successful remote login.

### Event Investigated

**Event ID 4624 — An account was successfully logged on**

### Key Evidence

- Event ID: **4624**
- Logon Type: **10**
- Target account: **Administrator**
- Authentication information

### Analysis

A successful authentication event was identified with **Logon Type 10**, which represents a RemoteInteractive logon and is associated with Remote Desktop Protocol (RDP).

### Evidence

![Successful RDP Login](screenshots/rdp-successful-login.png)

### Finding

> Event ID 4624 confirmed a successful authentication. The presence of Logon Type 10 identified the session as a RemoteInteractive/RDP logon involving the Administrator account.

### Investigation Assessment

The successful RDP logon was correlated with the previously observed failed authentication activity to determine whether the authentication sequence could represent a successful brute-force attempt.

---

# 3. Phishing-Based Malicious File Execution

## Objective

Investigate suspicious file execution following a phishing-based initial-access scenario.

### Event Investigated

**Sysmon Event ID 1 — Process Creation**

### Analysis

Sysmon process-creation telemetry was analyzed to identify the executable that was launched and understand the execution context.

The suspicious executable was:
C:\Users\Administrator\Pictures\best-cat.jpg.exe

The double extension .jpg.exe is suspicious because it can make an executable appear to be an image file.

Evidence 1
Finding

Sysmon Event ID 1 confirmed the creation of best-cat.jpg.exe from the user's Pictures directory. The .jpg.exe double extension was identified as a potential masquerading technique designed to make an executable appear to be an image.

Evidence 2 — Process Details

Finding:

The Sysmon event provided additional forensic information including the command line, process metadata, integrity level, parent process information, and file hashes. These artifacts can be used for process investigation, IOC extraction, and further threat hunting.

4. Network IOC — DNS Query Analysis
Objective
Investigate DNS activity associated with the suspicious executable and identify potential network-level indicators.

Event Investigated
Sysmon Event ID 22 — DNS Query

Analysis:
A Sysmon DNS Query event was identified in which the suspicious executable generated DNS activity.

The event provided information including:

Query name
Query status
Query results
Process ID
Process image
User
Process GUID
Evidence

Finding:
Sysmon Event ID 22 recorded DNS activity associated with the suspicious executable best-cat.jpg.exe. The DNS telemetry provided network-level evidence that could be correlated with the process responsible for generating the request.

IOC Investigation

The queried domain and any resolved IP address can be further investigated using threat-intelligence sources and correlated against other network and endpoint telemetry.

Note: A DNS query alone does not prove that a domain is malicious. The domain should be validated using additional threat-intelligence and investigation evidence.

5. User Account Persistence Investigation
Objective

Investigate suspicious account-management activity that could provide an attacker with persistent or elevated access.

5.1 New User Account Creation
Event Investigated

Event ID 4720 — A user account was created

Analysis:
Windows Security logs were analyzed for account-creation activity.

The investigation identified the creation of a new account named:

support
The event indicated that the account was created by:
Administrator
Evidence

Finding:
Windows Security Event ID 4720 confirmed the creation of a new user account named support. The event showed that the account was created by Administrator. The creation of an unexpected account was identified as a potential persistence mechanism and warranted further investigation.

5.2 Account Added to Administrators Group
Event Investigated
Event ID 4732 — A member was added to a security-enabled local group

Analysis:
The Windows Security log was examined for changes to local security-group membership.

The event showed activity involving the local:

Administrators
security group.
Evidence
Finding

Windows Security Event ID 4732 showed that a member was added to the local Administrators security group. Adding an account to a privileged local group can provide elevated permissions and may be used to establish or maintain persistence on a compromised system.

Investigation Conclusion

Correlation of the account-management events revealed potentially suspicious activity involving user-account creation and privileged group membership.

The sequence:

User Account Created
        ↓
Privileged Group Membership Change
        ↓
Potential Persistent Elevated Access

was treated as a potential persistence mechanism and would require further investigation and correlation with authentication, endpoint, and process telemetry.

Key Findings :
Investigation	Event ID	Finding
RDP Brute Force	4625	Repeated failed authentication attempts observed
Successful RDP Login	4624	Successful RemoteInteractive logon identified
Malicious File Execution	Sysmon 1	Suspicious .jpg.exe executable executed
DNS Investigation	Sysmon 22	DNS activity associated with suspicious executable
Account Creation	4720	New support user account created
Group Membership	4732	Account added to local Administrators group
SOC Analyst Investigation Workflow

The investigation followed a structured SOC workflow:

Detect
  ↓
Collect Event Evidence
  ↓
Analyze Event IDs
  ↓
Correlate Related Events
  ↓
Identify IOCs
  ↓
Assess Potential Impact
  ↓
Document Findings
  ↓
Recommend Response

If this activity were detected in a production environment, recommended actions would include:

Authentication Activity
Investigate the source IP associated with suspicious authentication attempts.
Determine whether the successful RDP login was authorized.
Review additional authentication events for the affected account.
Disable or secure compromised accounts if compromise is confirmed.
Malicious File
Isolate the affected endpoint if malicious execution is confirmed.
Collect and investigate the file hash.
Search the environment for the same hash or filename.
Review the process tree and child processes.
Investigate associated network connections.
Network IOC
Investigate the identified domain and IP address using threat intelligence.
Search DNS and proxy logs for additional activity involving the IOC.
Block confirmed malicious indicators where appropriate.
User Account Persistence
Validate whether the support account was legitimately created.
Review who created and modified the account.
Investigate its group memberships and privileges.
Disable/remove unauthorized accounts after appropriate incident-response procedures.
Skills Demonstrated
Windows Event Log Analysis
Windows Security Log Investigation
Sysmon Analysis
Event ID Investigation
RDP Brute-Force Detection
Authentication Analysis
Process Creation Analysis
Process Investigation
DNS Analysis
Network IOC Identification
User Account Persistence Detection
Privilege/Group Membership Investigation
Event Correlation
Incident Investigation
SOC Documentation
Conclusion

This investigation provided hands-on experience analyzing Windows Security and Sysmon telemetry to identify suspicious activity across authentication, process execution, network communication, and account management.

By correlating multiple Windows events, I was able to investigate potential brute-force activity, identify a successful RDP logon, analyze suspicious executable execution, investigate DNS activity, and identify potentially persistent user-account activity.

The investigation demonstrates a structured approach to Windows-based threat detection and SOC investigation.
