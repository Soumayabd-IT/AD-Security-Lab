# Smb rpc enumeration

## Théorie
> Rappel bref du principe. Détail complet dans [`docs/protocols/smb-rpc.md`](../../docs/protocols/smb-rpc.md)

## Prérequis
- Accès réseau : 
- Outils utilisés : 'netexec','smbclient'
- Comptes / permissions nécessaires : 

## Étapes pratiques

### 1. Initial SMB Discovery (no credentials)


```
netexec smb 192.168.10.7 -u'' -p''
```
*(capture d'écran : `../../screenshots/01-recon/01.png`)*

**Result:**



### 2. Attempting anonymous share enumeration 
```
netexec smb 192.168.10.7 -u'' -p'' --shares
```
*(capture d'écran : `../../screenshots/01-recon/02.png`)*

**Result:**

### 4.Authenticated enumeration (after obtaining credentials via LLMNR poisoning)
```
netexec smb 192.168.10.7 -u administrator -p 'MAMA1papa2@22' --shares
```
*(capture d'écran : `../../screenshots/01-recon/02.png`)*

`(Pwn3d!)` confirms full local admin rights on this machine.
Write access to `ADMIN$` and `C$` enables remote code execution (used later in Golden Ticket → psexec).

## Result
Anonymous enumeration completely blocked on this target — confirms correct hardening configuration. Authenticated enumeration reveals full share access including administrative shares, enabling lateral movement and remote execution.

## Detection & remediation
- **Detection**: Monitor for anonymous SMB connection attempts (Event ID 4625 with null session)
- **Remediation**: Keep RestrictAnonymous enabled; enforce SMB signing (already active here); disable SMBv1

## References
- MITRE ATT&CK T1135 — Network Share Discovery
- https://www.thehacker.recipes/ad/recon/smb
