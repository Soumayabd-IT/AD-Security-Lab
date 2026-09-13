# DCSync

## Theory
DCSync is an attack that abuses the Active Directory replication
protocol (DRSUAPI). Normally, only Domain Controllers replicate
the AD database between themselves. An account with replication
rights (DS-Replication-Get-Changes + DS-Replication-Get-Changes-All)
can request this replication and receive the NT hash of every account
in the domain — including krbtgt.
See [`docs/protocols/kerberos.md`](../../docs/protocols/kerberos.md)

## Prerequisites
- A domain account with replication rights
  (default: Domain Admins, Enterprise Admins, Domain Controllers)
- Obtained here via LLMNR poisoning →
  see [`attacks/02-credential-capture/llmnr-nbtns-poisoning.md`](../02-credential-capture/llmnr-nbtns-poisoning.md)
- Tool: `impacket-secretsdump`

## Practical steps

### 1. Run secretsdump against the DC
```bash
impacket-secretsdump bob.local/administrator:'MAMA1papa2@22'@192.168.10.7
```

### 2. Key output — local SAM hashes
These are the local machine accounts (not domain-wide).

### 3. Key output — domain credentials via DRSUAPI

### 4. Key output — Kerberos keys

The AES256 key for krbtgt enables forging tickets
that bypass RC4 detection rules.

## Result
Complete extraction of all domain account hashes via the DRSUAPI
replication method. The krbtgt NT hash
(`d25b2d64b57f0355c5400e6698289455`) and AES256 key are the
critical outputs — enabling:
- Golden Ticket forgery →
  see [`attacks/03-kerberos-attacks/golden-ticket.md`](../03-kerberos-attacks/golden-ticket.md)
- Pass-the-Hash for any domain account →
  see [`attacks/04-lateral-movement/pass-the-hash.md`](../04-lateral-movement/pass-the-hash.md)

## Detection & remediation
- **Detection**: Event ID 4662 on the DC — an object performed a
  replication operation. Alert on non-DC accounts triggering this
  event; also monitor for large volumes of 4662 events in a short
  timeframe (signature of secretsdump)
- **Remediation**: Restrict DS-Replication-Get-Changes rights
  strictly to Domain Controllers; audit ACLs regularly with
  BloodHound to detect accounts with unexpected replication rights

## References
- MITRE ATT&CK T1003.006 — DCSync
- https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync
