# Ldap enumeration

## Théorie
> Rappel bref du principe. Détail complet dans [`docs/protocols/ldap.md`](../../docs/protocols/ldap.md)

## Prérequis
- Accès réseau :  Network access to the target (port 389)
- Outils utilisés : `netexec`
- Comptes / permissions nécessaires : 

## Étapes pratiques

### 1. Attempting anonymous LDAP bind
```
netexec ldap 192.168.10.7 -u '' -p ''
```
*(capture d'écran : `../../screenshots/01-recon/01.png`)*
**Result:**
Anonymous LDAP bind rejected — directory queries require prior authentication.

### 2.  Authenticated LDAP enumeration (after obtaining credentials)
```
netexec ldap 192.168.10.7 -u administrator -p 'MAMA1papa2@22' --users
```
*(capture d'écran : `../../screenshots/01-recon/02.png`)*
**Result**
-Username- -Last PW Set- -BadPW-
Administrator 2026-06-22 10:42:13 0
Guest <never> 0
krbtgt 2026-07-30 16:35:59 0
jsmith 2026-08-11 03:04:54 0
tsmith 2026-07-30 16:51:58 6

## Résultat
Ce qu'on obtient à l'issue de l'attaque (hash, ticket, accès, credentials...).

## Détection & remédiation

## Result
Anonymous LDAP enumeration blocked. Authenticated enumeration reveals all domain users including the krbtgt service account — confirming targets for Kerberoasting and further attacks.

## Detection & remediation
- **Detection**: Monitor for bulk LDAP queries, especially filters targeting SPNs or all user accounts
- **Remediation**: Keep anonymous bind disabled; enforce LDAP signing and channel binding

## References
- MITRE ATT&CK T1087.002 — Domain Account Discovery
- https://www.thehacker.recipes/ad/recon/ldap
