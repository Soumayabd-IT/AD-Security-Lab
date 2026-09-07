# GPO & ACL

## Role in Active Directory
A **GPO (Group Policy Object)** is a set of configuration rules (system settings, scripts, restrictions...) automatically applied to users or computers. An **ACL (Access Control List)** defines who is allowed to do what on a given directory object (read, modify, delete, reset a password...). The two mechanisms control different aspects but often overlap in penetration testing: a misconfigured ACL can grant the right to modify a GPO, and a GPO can be hijacked to execute code on hundreds of machines at once.

## Key definitions

### GPO
- **GPC (Group Policy Container)**: the part of the GPO stored in the AD directory itself (metadata, links).
- **GPT (Group Policy Template)**: the part stored on disk, in the `SYSVOL` share of every domain controller (scripts, actual configuration files).
- **GPLink**: the attribute that links a GPO to a container (site, domain, or organizational unit).
- **Application order — LSDOU**: Local → Site → Domain → OU. The last rule applied (the most specific, at the OU level) wins in case of conflict, unless the "Enforced" option is set on a GPO higher up in the hierarchy.
- **Block Inheritance**: an option that prevents an OU from inheriting GPOs from its parents (except those marked "Enforced").

### ACL
- **Security Descriptor**: a structure attached to every AD object, containing its owner and its permissions.
- **DACL (Discretionary ACL)**: the list that defines who is allowed to do what (read, write, delete...) — the "permissions" in the everyday sense.
- **SACL (System ACL)**: the list that defines which actions are audited/logged on the object.
- **ACE (Access Control Entry)**: a single line within a DACL or SACL — "this user/group has this right, allowed or denied".
- **Commonly abused rights**: `GenericAll` (full control), `GenericWrite` (modify most attributes), `WriteOwner` (become the object's owner), `WriteDACL` (modify the permissions themselves), and extended rights such as `User-Force-Change-Password` or `DS-Replication-Get-Changes` (used for DCSync).

## How it works
1. A GPO is created and linked (`GPLink`) to a site, a domain, or an OU.
2. At startup/logon, the relevant machines/users download the applicable GPOs from `SYSVOL` and apply them following the LSDOU order.
3. In parallel, every AD object (user, group, the GPO itself...) has a DACL that determines who can read or modify it — including who is allowed to modify a given GPO.

## Exploited weaknesses
- A user with write access to a GPO (`GenericWrite`, `WriteProperty`) can inject a malicious startup script or scheduled task into it, automatically executed on every machine where the GPO is linked → [`attacks/05-privilege-escalation/gpo-abuse.md`](../../attacks/05-privilege-escalation/gpo-abuse.md)
- A misconfigured ACL (inherited, or granted by mistake) can give a low-privilege user a right such as `GenericAll` over a high-value account → full takeover of the target account → [`attacks/05-privilege-escalation/acl-abuse.md`](../../attacks/05-privilege-escalation/acl-abuse.md)
- The extended right `DS-Replication-Get-Changes` (normally reserved for domain controllers), granted by mistake, enables a DCSync attack → [`attacks/06-domain-dominance/dcsync.md`](../../attacks/06-domain-dominance/dcsync.md)
- These misconfigured permission chains are hard to spot by eye on a large domain — which is exactly what BloodHound maps automatically → [`attacks/05-privilege-escalation/bloodhound-mapping.md`](../../attacks/05-privilege-escalation/bloodhound-mapping.md)

## Defensive takeaways
- Regularly audit domain ACLs (via BloodHound or dedicated scripts) to spot unintended escalation paths
- Restrict GPO modification rights to delegated administrators only
- Monitor GPO modification events (Event ID 5136) and permission changes on sensitive objects
- Apply the principle of least privilege: avoid broad delegations (`GenericAll`) when a more precise right would suffice

## References
-
