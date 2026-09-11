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


### 3. Déclenchement de la requête côté victime
Sur la machine Windows, tentative de résolution d'un partage inexistant :

### 4. Capture du hash NTLMv2
*(capture d'écran : `../../screenshots/02-credential-capture/02.png`)*

Responder intercepte la requête LLMNR/mDNS et capture plusieurs hashs NTLMv2 pour le compte `Administrator` (un nouveau hash à chaque tentative, car le challenge change).
## Résultat
Ce qu'on obtient à l'issue de l'attaque (hash, ticket, accès, credentials...).

### 5. Cassage hors ligne avec Hashcat

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 5600 hash.txt passlist.txt (mon propre liste)
hashcat -m 5600 hash.txt --show
```
*(capture d'écran : `../../screenshots/02-credential-capture/03.png`)*
## Résultat
Obtention du mot de passe en clair du compte `Administrator` du domaine `BOB.local` — accès complet aux privilèges administratifs du domaine, sans jamais avoir eu d'identifiant de départ.

## Détection & remédiation
- **Détection** : surveiller le trafic LLMNR (UDP 5355) et NBT-NS (UDP 137) anormal sur le réseau — un outil comme Responder actif génère un volume de réponses inhabituel.
- **Remédiation** : désactiver LLMNR (GPO) et NBT-NS (configuration réseau) si non nécessaires ; forcer une politique de mots de passe robuste pour résister au cracking même en cas de capture.

## Références
-
