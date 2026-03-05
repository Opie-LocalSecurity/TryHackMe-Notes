# Active Directory Basics

## Why Windows Domains

- Centralize management of many users/computers via **Active Directory (AD)** on a **Domain Controller (DC)**.
- Benefits:
    - **Centralized identity**: create/modify users in one place.
    - **Centralized policy**: push security/configuration policies domain-wide.

## Core AD Concepts (AD DS Objects)

- **Security principals** (can be authenticated & assigned permissions): **Users**, **Machines (computer accounts)**, **Security Groups**.
- **Users**
    - Represent people or services (service accounts run apps with least privilege).
- **Machines**
    - Each joined computer gets `COMPUTERNAME$` account; passwords auto-rotate (long, random).
- **Security Groups**
    - Grant permissions to resources (files, printers, shares).
    - Key built-ins: Domain Admins, Server/Backup/Account Operators, Domain Users/Computers/Controllers.

## ADUC & Organizational Units (OUs)

- Manage via **Active Directory Users and Computers** (ADUC).
- **OUs** = containers to group users/computers for **policy application** and admin delegation.
    - Typical OU layout mirrors org structure (e.g., IT, Sales, Marketing).
    - Default containers: Builtin, Computers, Domain Controllers, Users, Managed Service Accounts.

### Groups vs OUs

- **OUs**: apply **policies**; an object belongs to **one** OU.
- **Security Groups**: grant **resource access**; a user can be in **many** groups.

## Managing Users

- **Delete protected OUs**: View → **Advanced Features** → OU Properties → uncheck “Protect object from accidental deletion” → delete.
- **Create/delete users** to match org changes.
- **Delegation** (granular admin):
    - Right-click OU → **Delegate Control…** → add user → choose tasks (e.g., reset passwords).
    - Example: delegate Sales OU password resets to `phillip`.
- **Password actions via PowerShell** (no ADUC rights needed):
    - Reset: `Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString)`
    - Force change at next logon: `Set-ADUser -Identity sophie -ChangePasswordAtLogon $true`

## Managing Computers

- New domain-joined devices land in **Computers** container by default.
- Create device-role OUs (recommended baseline):
    - **Workstations**, **Servers**, **Domain Controllers** (already exists).
- Move devices into appropriate OUs to target policies accurately.

## Group Policy Objects (GPOs)

- Configure via **Group Policy Management**.
- Workflow: create GPO under **Group Policy Objects** → **link** it to target OU(s)/domain.
- Scope & inheritance:
    - GPOs apply to the linked OU **and all child OUs**.
    - Use **Security Filtering** to narrow to specific users/computers.
- Example built-ins:
    - `Default Domain Policy` (e.g., password/lockout settings) linked at domain root.
    - `Default Domain Controllers Policy` linked to DCs OU.

### Editing Policies

- Edit path example for password length:
    - **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy** (e.g., set min length to 10).
- Each policy has an **Explain** tab for details.

### Distribution & Refresh

- Policies replicate via **SYSVOL** (`C:\Windows\SYSVOL\sysvol\`).
- Refresh can take up to ~2 hours; force with:
    - `gpupdate /force`

### GPO Scenarios

- **Restrict Control Panel (non-IT users)**:
    - New GPO: *Restrict Control Panel Access*
    - **User Configuration → Administrative Templates → Control Panel → Prohibit access… → Enabled**
    - Link to user OUs (e.g., Marketing, Management, Sales); **do not** link to IT.
- **Auto Lock Screen (all computers)**:
    - New GPO: *Auto Lock Screen*
    - **Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options → Interactive logon: Machine inactivity limit** = **5 minutes**
    - Link at **domain root** so Workstations/Servers/DCs inherit (user-only OUs ignore computer settings).

## Authentication in AD

- **Kerberos** (default, ticket-based):
    1. Client → KDC: AS-REQ (user + timestamp encrypted with key from password).
    2. KDC → Client: **TGT** (encrypted with **krbtgt** hash) + **Session Key**.
    3. Client → KDC: TGS-REQ (TGT + SPN + data encrypted with Session Key).
    4. KDC → Client: **TGS** + **Service Session Key** (TGS encrypted with service account hash).
    5. Client → Service: presents TGS; service decrypts and validates to grant access.
- **NetNTLM (NTLM)** (legacy, challenge–response):
    - Server sends challenge → client computes response using NT hash → server asks DC to verify (domain accounts) → match = authenticated.
    - Password/hash never sent on the wire.
    - For **local** accounts, server validates directly (uses local SAM).

## Multi-Domain Design: Trees, Forests, Trusts

- **Tree**: multiple domains in the **same namespace** (e.g., `thm.local` with `uk.thm.local`, `us.thm.local`).
    - Each domain has its own DCs and **Domain Admins**.
    - **Enterprise Admins**: admins across **all** domains in the enterprise.
- **Forest**: collection of one or more **trees/namespaces** (e.g., `thm.local` + `mht.local`) under one umbrella.
- **Trusts**: allow cross-domain authorization.
    - **One-way trust**: if A trusts B, **B’s users can access A** (when allowed).
    - **Two-way trust**: mutual access; default between domains in same tree/forest.
    - Trust **enables** authorization; it does **not** grant access by itself—admins still assign permissions.