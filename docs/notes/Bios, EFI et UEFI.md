---
share: true
revision: true
MOC:
  - "[[MOC_Perso_Informatique|MOC_Perso_Informatique]]"
tags:
  - Informatique
  - Infra
  - Bios
  - EFI
  - UEFI
---
---
share: true
revision: true
MOC: 
- "[[MOC_Perso_Informatique|MOC_Perso_Informatique]]"
tags:
- Informatique
- Infra
- Bios
- EFI
- UEFI
---
# Bios, EFI et UEFI
## Bios
Le **BIOS** (Basic Input/Output System) est un logiciel de bas niveau intégré à la carte mère de l'ordinateur. Fournit également des services de base comme POST Power-On Self Test (test du matériel faisant des bip en cas d'erreur), Initialisation du matériel comme clavier souris, Gestion du boot, interface utilisateur (souvent en faisant F2, Del, ou ESC), etc.

## EFI
Développée par Intel, **EFI** est une interface standard entre le système d'exploitation d'un ordinateur et son micrologiciel (firmware), elle remplace le BIOS.
Fonctionnalités supplémentaires : Interface graphique, Compatibilité des pilotes (permet d'initialiser les pilotes avant de charger l'OS), Gestion des partitions directement dans EFI, etc.

## UEFI
**UEFI** est l'évolution de l'EFI, standardisée par le forum UEFI. UEFI offre des améliorations et des extensions par rapport à l'EFI, incluant :
- **Sécurité** : UEFI inclut Secure Boot, une fonctionnalité de sécurité qui aide à prévenir l'exécution de logiciels malveillants au démarrage du système.
- **Compatibilité** : UEFI est compatible avec les anciens systèmes BIOS, permettant ainsi de fonctionner sur du matériel plus ancien.
- **Support de disques GPT** : UEFI supporte le GUID Partition Table (GPT), une méthode moderne de gestion des partitions de disque qui surpasse les limitations du MBR.