# AD Account Lockout Lab

Simulated Active Directory account lockout lab documenting the full help desk workflow — user creation, intentional lockout via failed authentication attempts, investigation using Windows Event Viewer, and resolution. Built to demonstrate real-world IT support and documentation skills.

---

## Tools Used
- Windows 11 VM (target machine)
- Kali Linux VM (attack machine)
- CrackMapExec (SMB authentication testing)
- Windows Event Viewer
- Local Security Policy (secpol.msc)
- Computer Management (Local Users and Groups)

---

## Lab Workflow
```
[ 1. Create User ]
       ↓
[ 2. Set Lockout Policy ]
  5 failed attempts → lock
       ↓
[ 3. Trigger Lockout from Kali ]
  crackmapexec smb → wrong password x5+
       ↓
[ 4. Confirm Lockout on Windows ]
  Event ID 4740 + Account Properties
       ↓
[ 5. Resolve — Unlock Account ]
  Computer Management → uncheck locked out
       ↓
[ 6. Document Ticket in Freshdesk ]
```

---

## Commands Used

**On Kali Linux — trigger the lockout:**
```bash
# Check if target is reachable
crackmapexec smb 192.168.56.102 -u testuser -p wrongpass1 --local-auth

# Confirm lockout status
crackmapexec smb 192.168.56.102 -u testuser -p wrongpass1 --local-auth
# Returns: STATUS_ACCOUNT_LOCKED_OUT
```

**On Windows — enable SMB and firewall rules:**
```
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=yes
netsh advfirewall firewall add rule name="Allow SMB" dir=in action=allow protocol=TCP localport=445
```

**On Windows — verify IP:**
```
ipconfig
```

---

## Evidence

### Step 1 — User Created
testuser created in Computer Management with description "AD Lab"

### Step 2 — Lockout Policy Configured
- Threshold: 5 invalid logon attempts
- Duration: 10 minutes
- Reset counter after: 10 minutes

### Step 3 — Lockout Triggered from Kali
CrackMapExec returned STATUS_ACCOUNT_LOCKED_OUT confirming the account was locked after repeated failed SMB authentication attempts from 192.168.56.102

### Step 4 — Lockout Confirmed on Windows
- testuser Properties showed "Account is locked out" checkbox checked
- Event Viewer logged Event ID 4740 — User Account Management
- Account Name: testuser | Security ID: Jameel\testuser
- Logged: 3/24/2026 1:38:46 PM

### Step 5 — Account Unlocked
Unchecked "Account is locked out" in testuser Properties
Account restored to active status — checkbox greyed out confirming resolution

---

## Key Event IDs Referenced
| Event ID | Description |
|----------|-------------|
| 4625 | Failed logon attempt |
| 4740 | User account locked out |
| 4767 | User account unlocked |

---

## Status
✅ Completed — 3/24/2026
