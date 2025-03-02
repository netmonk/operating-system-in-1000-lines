---
title: Bien démarrer
---

# Bien démarrer

Le contenu de ce livre s'adresse à des gens familiers des systèmes d'exploitation Unix et assimilés comme macOS ou Linux (Ubuntu). Pour les utilisateurs des Windows, il faut installer Windows Subsystem for Linux (WSL2) et suivre les instructions associées à Ubuntu.

## Installation des outils de developement

### macOS 

Installez [Homebrew](https://brew.sh) et executez cette command pour installer tous les outils requis:

```
brew install llvm lld qemu
```

Ensuite, ajoutez binutils de LLVM à votre `PATH` shell:

```
$ export PATH="$PATH:$(brew --prefix)/opt/llvm/bin"
$ which llvm-objcopy
/opt/homebrew/opt/llvm/bin/llvm-objcopy
```

### Ubuntu

Installez les paquets suivant avec apt:

```
sudo apt update && sudo apt install -y clang llvm lld qemu-system-riscv32 curl
```

Télechargez OpenSBI (l'équivalent du BIOS/UEFI pour PCs):

```
curl -LO https://github.com/qemu/qemu/raw/v8.0.4/pc-bios/opensbi-riscv32-generic-fw_dynamic.bin
```

> [!ATTENTION]
>
> Pour QEMU, il est nécessaire que `opensbi-riscv32-generic-fw_dynamic.bin` soit dans le répertoire courant d'où vous lancez QEMU. S'il n'est pas présent vous obtiendrez l'erreur suivante: 
>
> ```
> qemu-system-riscv32: Unable to load the RISC-V firmware "opensbi-riscv32-generic-fw_dynamic.bin"
> ```

### Pour les autres OSes

Si vous utilisez un autre OS que ceux précédents, installez les outils suivants: 

- `bash`: La ligne de command shell (c'est souvent déjà installé).
- `tar`: C'est aussi installé par défaut, préférez la version GNU et non BSD.
- `clang`: Le compilateur C. Assurez-vous qu'il supporte l'architecture 32-bit RISC-V CPU (voir en dessous).
- `lld`: Le générateur de liens LLVM, utile pour fabriquer un fichier exécutable à partir d'objets compilés.
- `llvm-objcopy`: Object file editor. It comes with the LLVM package (typically `llvm` package).
- `llvm-objdump`: Un dessassembleur. 
- `llvm-readelf`: Un analyseur de fichiers au format ELF.
- `qemu-system-riscv32`: L'Emulateur 32-bit RISC-V CPU. Il fait parti du paquet QEMU.

> [!ASTUCE]
>
> Pour vérifier si `clang` supporte l'architecture 32-bit RISC-V CPU, executez cette commande:
>
> ```
> $ clang -print-targets | grep riscv32
>     riscv32     - 32-bit RISC-V
> ```
>
> Vous devriez voir `riscv32`. La version pré-installée sur macOS ne montre pas ce résultat. Pour cette raison, il faut installer la version alternative contenue dans le paquet `llvm` de HomeBrew´s

## Initialiser un dépot GIT (optionel)

Si vous utilisez un dépot `git`, configurer le fichier `.gitignore` ainsi: 

```gitignore [.gitignore]
/disk/*
!/disk/.gitkeep
*.map
*.tar
*.o
*.elf
*.bin
*.log
*.pcap
```

Tout est bon! Allons construire notre premier système d'exploitation!
