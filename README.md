# AD Security Lab

Documentation d'un lab personnel de sécurité Active Directory : mise en place de l'environnement, théorie des protocoles, et attaques pratiquées (avec captures d'écran et remédiations).

## Objectifs du projet

- Comprendre en profondeur le fonctionnement d'Active Directory et de ses protocoles (Kerberos, LDAP, SMB/RPC, NTLM...)
- Pratiquer et documenter des techniques d'attaque réelles dans un environnement contrôlé
- Relier attaque et défense (détection, remédiation)

## Structure du repo

```
AD-Security-Lab/
├── docs/
│   ├── protocols/      # Théorie des protocoles AD, indépendante des attaques
│   └── lab-setup/       # Comment le lab a été monté (topologie, config réseau, DC)
├── attacks/
│   ├── 01-recon/
│   ├── 02-credential-capture/
│   ├── 03-kerberos-attacks/
│   ├── 04-lateral-movement/
│   ├── 05-privilege-escalation/
│   └── 06-domain-dominance/
├── screenshots/          # Captures d'écran, même arborescence que attacks/
└── detection/            # Règles / notes de détection (Splunk, Sysmon, MITRE ATT&CK)
```

Les dossiers `attacks/` sont numérotés pour suivre une kill chain réaliste : reconnaissance → capture d'identifiants → attaques Kerberos → mouvement latéral → élévation de privilèges → prise de contrôle du domaine.

Chaque fiche d'attaque suit le même format : **Théorie → Prérequis → Étapes pratiques → Résultat → Détection & remédiation → Références**.

## Environnement du lab

À détailler dans [`docs/lab-setup/topology.md`](docs/lab-setup/topology.md).

## Avertissement

Ce lab est réalisé dans un environnement personnel et isolé, à des fins d'apprentissage uniquement.
