# Ce que nous allons implémenter

Avant de commencer à construire un OS, passons rapidement en revue les fonctionnalités que nous allons implémenter.

## Fonctionnalités dans l'OS en 1K LoC

Dans ce livre, nous allons implémenter les fonctionnalités majeures suivantes :

- **Multitâche** : Permet de basculer entre plusieurs processus afin que plusieurs applications puissent partager le CPU.
- **Gestionnaire d'exceptions** : Gère les événements nécessitant une intervention de l'OS, comme les instructions illégales.
- **Pagination** : Fournit un espace mémoire isolé pour chaque application.
- **Appels système** : Permet aux applications d'accéder aux fonctionnalités du noyau.
- **Pilotes de périphériques** : Abstrait les fonctionnalités matérielles, comme la lecture/écriture sur disque.
- **Système de fichiers** : Gère les fichiers stockés sur le disque.
- **Shell en ligne de commande** : Interface utilisateur permettant aux humains d'interagir avec le système.

## Fonctionnalités non implémentées

Les fonctionnalités majeures suivantes ne sont pas implémentées dans ce livre :

- **Gestion des interruptions** : À la place, nous utiliserons une méthode de scrutation (vérification périodique des nouvelles données sur les périphériques), également connue sous le nom de busy waiting.
- **Gestion des timers** : Le multitâche préemptif n'est pas implémenté. Nous utiliserons un multitâche coopératif, où chaque processus cède volontairement le CPU.
- **Communication inter-processus** : Des fonctionnalités comme les pipes, les sockets UNIX et la mémoire partagée ne sont pas implémentées.
- **Support multi-processeur** : Seul un processeur unique est pris en charge.

## Structure du code source

Nous allons construire notre OS progressivement à partir de zéro, et la structure finale des fichiers ressemblera à ceci :
```
├── disk/     - File system contents
├── common.c  - Kernel/user common library: printf, memset, ...
├── common.h  - Kernel/user common library: definitions of structs and constants
├── kernel.c  - Kernel: process management, system calls, device drivers, file system
├── kernel.h  - Kernel: definitions of structs and constants
├── kernel.ld - Kernel: linker script (memory layout definition)
├── shell.c   - Command-line shell
├── user.c    - User library: functions for system calls
├── user.h    - User library: definitions of structs and constants
├── user.ld   - User: linker script (memory layout definition)
└── run.sh    - Build script
```

> [!TIP]
>
> Dans ce livre, "espace utilisateur" est parfois abrégé en "utilisateur". Considérez cela comme "applications" et ne le confondez pas avec un "compte utilisateur" !

