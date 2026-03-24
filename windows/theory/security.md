---
icon: lock-keyhole
---

# Security

## <mark style="color:$primary;">ABOUT</mark>

Here I would write about <mark style="color:purple;">**security mechanisms**</mark> in Windows. Since Windows theory is very broad, and to avoid making the theory page bloated, I would write here about the security side of Windows.&#x20;

## <mark style="color:$primary;">AUTHENTICATION</mark>

<mark style="color:red;">**Authentication**</mark> is process of verifying <mark style="color:purple;">**who someone is.**</mark>

<figure><img src="../../.gitbook/assets/win_authproc_image.png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$primary;">AUTHORIZATION</mark>

<mark style="color:red;">**Authorization**</mark> is process of determining <mark style="color:purple;">**what someone is allowed to do**</mark><mark style="color:purple;">.</mark>

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption><p><a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-principals"><strong>[SOURCE]</strong></a></p></figcaption></figure>

<mark style="color:red;">**Security principals**</mark> are <mark style="color:purple;">**anything that can be authenticated**</mark> by the Windows operating system, including user and computer accounts, processes that run in the security context or another user/computer account, or the security groups that these accounts belong to. Every single security principal is identified by a unique <mark style="color:red;">**Security Identifier (SID)**</mark>. When a security principal is created, it is assigned a SID which remains assigned to that principal for its lifetime.

1. <mark style="color:blue;">**User Authentication and Access Token Generation.**</mark> A user logs into the system or attempts to access a resource. The system authenticates the user and generates an **access token**.
2. <mark style="color:blue;">**Request to Access a Securable Object.**</mark> The user attempts to access a **securable object**. The object has a **security descriptor** that includes its **Access Control List (ACL)**. The ACL contains **Access Control Entries (ACEs)**, which define which SIDs (users or groups) have specific permissions to access the object.
3. <mark style="color:blue;">**Comparison of Access Token and Security Descriptor.**</mark> The system compares the user's **access token** (SIDs and privileges) with the object's **ACEs** in the security descriptor. Permissions such as **Read**, **Write**, **Execute**, etc., are checked against the ACEs to determine whether the user has the required access rights.
4. <mark style="color:blue;">**Access Decision.**</mark>

## <mark style="color:$primary;">SAM</mark>

<mark style="color:red;">**The Security Accounts Manager**</mark>**&#x20;is a&#x20;**<mark style="color:purple;">**database file**</mark>**&#x20;in the Microsoft Windows operating system that contains usernames and passwords.** The SAM is available in different versions of Windows, including XP, Vista, 7, 8.1, 10 and 11. Each user account can be assigned a local area network (LAN) and a password would be hashed and stored in the SAM. The passwords hashes are stored in <mark style="color:green;">**HKEY\_LOCAL\_MACHINE\SAM**</mark><mark style="color:blue;">**,**</mark> but the access to it is restricted. **HKLM/SAM** and **SYSTEM** privileges are required for accessing it.

## <mark style="color:$primary;">SID</mark>

<mark style="color:red;">**Security Identifier (SID)**</mark> is <mark style="color:purple;">**unique, system-generated identifier**</mark> for each security principal on a system. Even identical users are distinguished by their SIDs, which determine their permissions. SIDs are stored in the security database and included in access tokens to manage user actions.&#x20;

A SID consists of the <mark style="color:red;">**Identifier Authority**</mark> and the <mark style="color:red;">**Relative ID (RID)**</mark>. In an **Active Directory (AD)** domain environment, the SID also includes the domain SID.

#### SID Structure:

<mark style="color:blue;">`(SID)-(revision level)-(identifier-authority)-(subauthority1)-(subauthority2)-(etc)`</mark>

<mark style="color:blue;">`S-1-5-21-674899381-4069889467-2080702030-1002`</mark>

| **Number**                                         | **Meaning**              | **Description**                                                                                                                                                                                                                             |
| -------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:blue;">**S**</mark>             | **SID**                  | Indicates the string is a SID.                                                                                                                                                                                                              |
| <mark style="color:blue;">**1**</mark>             | **Revision Level**       | Always 1, showing the SID format version.                                                                                                                                                                                                   |
| <mark style="color:blue;">**5**</mark>             | **Identifier Authority** | Identifies the authority (e.g., computer or network) that created the SID.                                                                                                                                                                  |
| <mark style="color:blue;">**21**</mark>            | **Subauthority 1**       | Shows the user's relation or group to the authority that created the SID.                                                                                                                                                                   |
| <mark style="color:blue;">**674899381-...**</mark> | **Subauthority 2**       | Identifies the specific computer or domain that created the SID.                                                                                                                                                                            |
| <mark style="color:blue;">**1002**</mark>          | **Subauthority 3 (RID)** | <p>Differentiates accounts, indicating roles like user, guest, or administrator. <br><mark style="color:orange;"><strong>(If you enumerated previous value via enum, you can bruteforce RID to get a lot objects SID's)</strong></mark></p> |

## <mark style="color:$primary;">LSASS</mark>

<mark style="color:red;">**Local Security Authority Subsystem Service (LSASS)**</mark> is a <mark style="color:purple;">**collection of many modules**</mark> and has access to all authentication processes that can be found in <mark style="color:green;">`%SystemRoot%\System32\Lsass.exe`</mark>. <mark style="color:green;">**`lsass.exe`**</mark> is the <mark style="color:purple;">**process**</mark> that is responsible for **enforcing the security policy on Windows systems**.&#x20;

When a user successfully logs in, LSASS constructs a <mark style="color:yellow;">**full access token**</mark> that contains the user SID, group SIDs, SIDHistory (if any), assigned privileges, and integrity level. This access token reflects the user’s effective identity and security context, and it gets attached to every process the user runs.

If the user is part of the Administrators group and UAC is enabled, Windows will split the LSASS-issued token into two versions:

* <mark style="color:yellow;">**Standard Token**</mark> (medium integrity) with filtered privileges and admin groups marked as deny-only
* <mark style="color:yellow;">**Elevated Token**</mark> (high integrity) that retains full admin privileges

This token split ensures users don’t run with full admin rights by default, supporting the principle of least privilege while still allowing elevation when needed.

All events associated with this process (logon/logoff attempts, etc.) are logged within the **Windows Security Log**. <mark style="color:orange;">**LSASS is an extremely high-value target**</mark> as several tools exist to extract both cleartext and hashed credentials stored in memory by this process.

### <mark style="color:blue;">Secrets</mark>

**LSA Secrets** are <mark style="color:purple;">**sensitive data**</mark> stored by the **Local Security Authority** (LSA) on Windows systems. Key Types of LSASS Secrets are:

1. <mark style="color:orange;">**Hashes**</mark>\
   LSASS stores password hashes for logged-in users. These can be used in attacks like **Pass-the-Hash** to authenticate without knowing the plain-text password.
2. <mark style="color:orange;">**Kerberos Tickets**</mark>\
   LSASS holds Kerberos tickets (TGTs and service tickets) used in Active Directory environments to authenticate users across the network.
3. <mark style="color:orange;">**Plain-Text Passwords**</mark>\
   In some cases, LSASS may store plain-text passwords or reversible encryption keys for convenience in single sign-on (SSO) scenarios.
4. <mark style="color:orange;">**Security Tokens**</mark>\
   LSASS maintains security tokens that represent a user’s session and define what resources the user can access.

This data is stored in the Windows Registry at <mark style="color:green;">**`HKEY_LOCAL_MACHINE\Security\Policy\Secrets`**</mark>

## <mark style="color:$primary;">NTFS</mark>

### <mark style="color:blue;">Permission</mark>

| Permission Type                                           | Description                                                                                             |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| <mark style="color:blue;">**Full Control**</mark>         | Allows reading, writing, changing, and deleting files/folders.                                          |
| <mark style="color:blue;">**Modify**</mark>               | Allows reading, writing, and deleting files/folders.                                                    |
| <mark style="color:blue;">**List Folder Contents**</mark> | Allows viewing and listing folders/subfolders and executing files. Folders inherit this permission.     |
| <mark style="color:blue;">**Read and Execute**</mark>     | Allows viewing, listing files/subfolders, and executing files. Inherited by both files and folders.     |
| <mark style="color:blue;">**Write**</mark>                | Allows adding files to folders and writing to a file.                                                   |
| <mark style="color:blue;">**Read**</mark>                 | Allows viewing and listing folders/subfolders and reading file contents.                                |
| <mark style="color:blue;">**Traverse Folder**</mark>      | Allows moving through folders to access files, even without permission to list or view folder contents. |

### <mark style="color:blue;">Inheritance</mark>

<mark style="color:red;">**Inheritance**</mark> in NTFS allows <mark style="color:purple;">**permissions to propagate from a parent folder to its subfolders and files**</mark>. This ensures consistent permissions throughout a directory structure. Modifiers like <mark style="color:blue;">**`(CI)`**</mark>, <mark style="color:blue;">**`(OI)`**</mark>, and <mark style="color:blue;">**`(IO)`**</mark> define how permissions are inherited by child objects.

#### **Inheritance Modifiers**

| Modifier                                  | Meaning                              | Explanation                                                                 |
| ----------------------------------------- | ------------------------------------ | --------------------------------------------------------------------------- |
| <mark style="color:blue;">**`CI`**</mark> | **Container Inherit**                | Applies permissions to subfolders (but not files) within the parent folder. |
| <mark style="color:blue;">**`OI`**</mark> | **Object Inherit**                   | Applies permissions to files (but not subfolders) within the parent folder. |
| <mark style="color:blue;">**`IO`**</mark> | **Inherit Only**                     | Ensures permissions are inherited but not applied directly to the parent.   |
| <mark style="color:blue;">**`NP`**</mark> | **Do Not Propagate Inherit**         | Prevents permissions from propagating further to child objects.             |
| <mark style="color:blue;">**`I`**</mark>  | **Permission Inherited from Parent** | Indicates that the permission was inherited from a parent container.        |

#### **Basic Access Permissions**

<table><thead><tr><th>Symbol</th><th width="286">Permission</th><th>Description</th></tr></thead><tbody><tr><td><mark style="color:blue;"><strong><code>F</code></strong></mark></td><td><strong>Full Access</strong></td><td>Grants all permissions, including modifying and deleting.</td></tr><tr><td><mark style="color:blue;"><strong><code>D</code></strong></mark></td><td><strong>Delete Access</strong></td><td>Allows deletion of the file or folder.</td></tr><tr><td><mark style="color:blue;"><strong><code>N</code></strong></mark></td><td><strong>No Access</strong></td><td>Denies all access to the file or folder.</td></tr><tr><td><mark style="color:blue;"><strong><code>M</code></strong></mark></td><td><strong>Modify Access</strong></td><td>Allows reading, writing, and deleting content.</td></tr><tr><td><mark style="color:blue;"><strong><code>RX</code></strong></mark></td><td><strong>Read &#x26; Execute</strong></td><td>Allows reading and executing files.</td></tr><tr><td><mark style="color:blue;"><strong><code>R</code></strong></mark></td><td><strong>Read-Only Access</strong></td><td>Permits viewing and listing of contents.</td></tr><tr><td><mark style="color:blue;"><strong><code>W</code></strong></mark></td><td><strong>Write-Only Access</strong></td><td>Allows adding new files and data but not reading.</td></tr></tbody></table>

## <mark style="color:$primary;">NTLM</mark>

<mark style="color:red;">**NTLM (NT LAN Manager)**</mark> is a <mark style="color:purple;">**challenge-response authentication protocol**</mark> used by Microsoft for securing user credentials during authentication—primarily in older Windows environments or for backwards compatibility.

### <mark style="color:blue;">Auth Flow</mark>

> NTLM uses a 3-step <mark style="color:yellow;">**challenge-response mechanism**</mark> to authenticate a user without transmitting their password over the network:

#### **Negotiate**

Client sends a <mark style="color:green;">`NEGOTIATE_MESSAGE`</mark> to the server indicating it supports NTLM.

#### **Challenge**

Server replies with a <mark style="color:green;">`CHALLENGE_MESSAGE`</mark> containing randomly generated **8-byte nonce** (server challenge) and target information (like domain name, server name, etc.).

#### **Authenticate**

Client uses the server challenge and its own password-derived hash to calculate a **response** (using DES/MD4 depending on NTLM version).

Sends an <mark style="color:green;">`AUTHENTICATE_MESSAGE`</mark> containing:

* The username
* Domain
* **LM/NTLM response**
* And optionally a session key

## <mark style="color:$primary;">ACL</mark>

<figure><img src="../../.gitbook/assets/ACL SCHEME.png" alt="" width="563"><figcaption><p><strong>ACL Structure</strong></p></figcaption></figure>

<mark style="color:red;">**Access Control List (ACL)**</mark> is a <mark style="color:purple;">**list of rules**</mark> that specifies which users or groups are allowed to access an object, such as a file or folder, and what actions they can perform (e.g., read, write, or execute).

#### <mark style="color:$primary;">**DACL**</mark>

<mark style="color:red;">**Discretionary Access Control List (DACL)**</mark> is a type of ACL that <mark style="color:purple;">**defines the permissions for users and groups to access an object**</mark>. It controls what actions can be performed on the object by each user or group, and the owner of the object can modify the DACL.

#### <mark style="color:$primary;">**SACL**</mark>

<mark style="color:red;">**System Access Control List (SACL)**</mark>, on the other hand, <mark style="color:purple;">**specifies what events related to an object should be logged for auditing purposes**</mark>. It helps track access attempts, whether successful or not.

#### <mark style="color:$primary;">**ACE**</mark>

<mark style="color:red;">**Access Control Entry (ACE)**</mark> is an <mark style="color:purple;">**individual entry in an ACL**</mark> that defines which users, groups, or processes have access to a file or to execute a process, for example. Each ACE lists the access rights granted or denied to a principal (user or group).

Each <mark style="color:red;">**ACE**</mark> is made up of the following <mark style="color:purple;">**four**</mark> components:

1. <mark style="color:red;">**Security identifier (SID)**</mark> of the user/group that has access to the object (or principal name graphically).
2. <mark style="color:red;">**Flag**</mark> <mark style="color:purple;">**denoting the type of ACE**</mark> (access denied, allowed, or system audit ACE).
3. <mark style="color:purple;">**Set of flags**</mark> that specify whether or not child containers/objects can inherit the given ACE entry from the primary or parent object.
4. <mark style="color:red;">**Access mask**</mark> which is a 32-bit value that defines the rights granted to an object.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Image from <a href="https://academy.hackthebox.com/"><strong>HTBA</strong></a></p></figcaption></figure>

## <mark style="color:$primary;">DPAPI</mark>

<mark style="color:red;">**Data Protection Application Programming Interface**</mark> is set of API which Windows uses for the symmetric encryption of asymmetric private keys and used by various third-party applications like:

* **Internet Explorer**
* **Google Chrome**
* **Outlook**
* **Remote Desktop Connection**
* **Credential Manager**

## <mark style="color:$primary;">UAC</mark>

<mark style="color:red;">**User Account Control (UAC)**</mark> is a <mark style="color:purple;">**security feature**</mark> in Windows to **prevent malware from running or manipulating processes** that could damage the computer or its contents.&#x20;

Applications and tasks always run under the security context of a non-administrator account unless an administrator explicitly authorizes these applications/tasks to have administrator-level access to the system to run. It is a convenience feature that protects administrators from unintended changes but is not considered a security boundary.

> <mark style="color:orange;">**Administrator user is given 2 tokens: a standard user key, to perform regular actions as regular level, and one with the admin privileges.**</mark>

User can sign into their system using a standard account. When processes run under a standard user token, they operate with the permissions assigned to a regular user. However, certain applications need elevated privileges to function properly, and UAC allows them to receive additional access rights as needed.

The default <mark style="color:yellow;">**RID**</mark> <mark style="color:yellow;">**500**</mark> administrator account always operates at the high mandatory level. With <mark style="color:red;">**Admin Approval Mode (AAM)**</mark> enabled, any new admin accounts we create will operate at the medium mandatory level by default and be assigned two separate access tokens upon logging in. In the example below, the user account <mark style="color:green;">`ven17`</mark> is in the administrators group, but <mark style="color:green;">`cmd.exe`</mark> is currently running in the context of their unprivileged access token.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><strong>UAC Strucure</strong> <a href="https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works"><strong>[SOURCE]</strong></a></p></figcaption></figure>

## <mark style="color:$primary;">PERMISSION, RIGHT, PRIVILEGE</mark>

#### <mark style="color:orange;">**Permission**</mark>

<mark style="color:red;">**Permission**</mark> in Windows <mark style="color:purple;">**controls access to a specific object**</mark> such as a file, folder, registry key, or service. Permissions are stored inside a **Discretionary Access Control List (DACL)** attached to the object. Each permission is represented by an **Access Control Entry (ACE)** that maps a **Security Identifier (SID**) to a set of allowed or denied actions like <mark style="color:green;">**Read**</mark>, <mark style="color:green;">**Write**</mark>, <mark style="color:green;">**Execute**</mark>, or <mark style="color:green;">**Delete**</mark>. Permissions are evaluated every time a user or process attempts to access an object, during what is called the Access Check.

#### <mark style="color:orange;">Privilege</mark>

<mark style="color:red;">**Privilege**</mark> is a special ability tied to an access token that <mark style="color:purple;">**allows a user or process to perform sensitive system operations**</mark>, often related to kernel-level functions. Examples include **SeDebugPrivilege** (debugging other processes), **SeBackupPrivilege** (reading any file regardless of permissions), or **SeImpersonatePrivilege** (impersonating another user's security context). Privileges are checked only when a privileged action is requested, not at regular file or resource access.

#### <mark style="color:orange;">Right</mark>

<mark style="color:red;">**Right**</mark> is a system-wide rule that determines <mark style="color:purple;">**whether a user or group is allowed to perform a general action on the system**</mark>, like logging on locally, logging on over RDP, or shutting down the computer. Rights are managed through security policy (Local Security Policy or Group Policy) and are checked only during the logon phase, when the system validates if a user is permitted to log in in a specific way. Rights are not evaluated when accessing objects like files or registry keys later.

## <mark style="color:$primary;">ACCESS FLOW</mark>

#### <mark style="color:orange;">**Authentication Phase**</mark>

* User presents credentials.
* LSASS authenticates using Kerberos/NTLM.
* If successful → LSASS creates **Logon Session** and <mark style="color:red;">**Primary Access Token**</mark> (SID + Groups + Privileges).

#### <mark style="color:orange;">**Process Creation Phase**</mark>

* User launches a process.
* <mark style="color:purple;">**Primary Access Token is assigned**</mark> to the new process.
* All new <mark style="color:purple;">**child processes inherit the parent’s token**</mark> unless overridden.

#### <mark style="color:orange;">**Accessing a Securable Object (file, registry key, service)**</mark>

* Process requests an operation (read, write, delete, start service, etc.).
* **Access Check Begins**
  * OS reads process's **Access Token**.
  * OS reads object's **Security Descriptor** (contains DACL/SACL).

#### <mark style="color:orange;">**DACL Evaluation**</mark>

* Compares process's **SIDs** to object's **ACEs**.
* Deny ACEs evaluated **first**.
* Allow ACEs evaluated **second**.
* Requested permissions compared to allowed permissions.

#### <mark style="color:orange;">**Privilege Check (only if needed)**</mark>

* If action requires a special Privilege (e.g., <mark style="color:green;">`SeBackupPrivilege`</mark>, <mark style="color:green;">`SeDebugPrivilege`</mark>),
* OS checks if token has it and if it's enabled.

#### <mark style="color:orange;">**Access Granted or Denied**</mark>

* If SID + ACE match and permissions sufficient → <mark style="color:green;">**Access Granted**</mark>.
* Otherwise → <mark style="color:red;">**Access Denied**</mark>.

