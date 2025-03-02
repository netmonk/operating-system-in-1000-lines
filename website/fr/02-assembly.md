# Découverte RISC-V

Tout comme les navigateurs web masquent les différences entre Windows/macOS/Linux, les systèmes d'exploitation masquent les différences entre les processeurs. En d'autres termes, un système d'exploitation est un programme qui contrôle le processeur afin de fournir une couche d'abstraction pour les applications.

Dans ce livre, j'ai choisi RISC-V comme architecture cible car :

- [La spécification](https://riscv.org/technical/specifications/) est simple et adaptée aux débutants.
- C'est une architecture de jeu d'instructions (ISA) en plein essor ces dernières années, aux côtés de x86 et Arm.
- Les choix de conception sont bien documentés dans la spécification et sont intéressants à lire.

Nous allons écrire un système d'exploitation pour **RISC-V 32 bits**. Bien sûr, il est possible d'écrire pour RISC-V 64 bits avec seulement quelques modifications. Cependant, une largeur de bits plus importante rend les choses légèrement plus complexes, et les adresses plus longues sont fastidieuses à lire.

## Machine virtuelle QEMU

Les ordinateurs sont composés de divers périphériques : processeur, mémoire, cartes réseau, disques durs, etc. Par exemple, bien que les iPhones et les Raspberry Pis utilisent des processeurs Arm, il est naturel de les considérer comme des ordinateurs différents.

Dans ce livre, nous utilisons la machine virtuelle `virt` de QEMU ([documentation](https://www.qemu.org/docs/master/system/riscv/virt.html)) car :

- Même si elle n'existe pas dans le monde réel, elle est simple et très proche des périphériques réels.
- Vous pouvez l'émuler sur QEMU gratuitement, sans avoir besoin d'acheter du matériel physique.
- En cas de problèmes de débogage, vous pouvez lire le code source de QEMU ou attacher un débogueur au processus QEMU pour comprendre ce qui ne fonctionne pas.

## Introduction à l'assembleur RISC-V

RISC-V, ou ISA RISC-V (Instruction Set Architecture), définit les instructions que le processeur peut exécuter. C'est similaire aux API ou aux spécifications des langages de programmation pour les développeurs. Lorsque vous écrivez un programme en C, le compilateur le traduit en assembleur RISC-V. Malheureusement, pour écrire un système d'exploitation, il est nécessaire d'écrire un peu de code assembleur. Mais ne vous inquiétez pas ! L'assembleur n'est pas aussi difficile que vous pourriez le penser.

> [!ASTUCE]
>
> **Essayez Compiler Explorer !**
>
> Un outil utile pour apprendre l'assembleur est [Compiler Explorer](https://godbolt.org/), un compilateur en ligne. En tapant du code C, il affiche le code assembleur correspondant.
>
> Par défaut, Compiler Explorer utilise l'assembleur x86-64. Spécifiez `RISC-V rv32gc clang (trunk)` dans le panneau de droite pour obtenir l'assembleur RISC-V 32 bits.
>
> Il est également intéressant d'expérimenter avec des options d'optimisation comme `-O0` (optimisation désactivée) ou `-O2` (optimisation de niveau 2) et d'observer les différences dans l'assembleur.

### Bases du langage assembleur

Le langage assembleur est une représentation (presque) directe du code machine. Jetons un coup d'œil à un exemple simple :

```asm
addi a0, a1, 123
```

En général, chaque ligne de code assembleur correspond à une seule instruction. La première colonne (`addi`) est le nom de l'instruction, également appelé *opcode*. Les colonnes suivantes (`a0, a1, 123`) sont les *opérandes*, les arguments de l'instruction. Dans ce cas, l'instruction `addi` ajoute la valeur `123` à la valeur du registre `a1` et stocke le résultat dans le registre `a0`.

### Registres

Les registres sont comme des variables temporaires dans le processeur, et ils sont beaucoup plus rapides que la mémoire. Le processeur lit les données de la mémoire dans les registres, effectue des opérations arithmétiques sur ces registres, puis écrit les résultats en mémoire ou dans d'autres registres.

Voici quelques registres courants en RISC-V :

| Registre | Nom ABI (alias) | Description |
|---| -------- | ----------- |
| `pc` | `pc`       | Compteur de programme (adresse de la prochaine instruction) |
| `x0` |`zero`     | Valeur constante zéro (toujours lu comme zéro) |
| `x1` |`ra`         | Adresse de retour |
| `x2` |`sp`         | Pointeur de pile |
| `x5` - `x7` | `t0` - `t2` | Registres temporaires |
| `x8` | `fp`      | Pointeur de cadre de pile |
| `x10` - `x11` | `a0` - `a1`  | Arguments de fonction/valeurs de retour |
| `x12` - `x17` | `a2` - `a7`  | Arguments de fonction |
| `x18` - `x27` | `s0` - `s11` | Registres temporaires sauvegardés entre appels |
| `x28` - `x31` | `t3` - `t6` | Registres temporaires |

> [!ASTUCE]
>
> **Convention d'appel :**
>
> En général, vous pouvez utiliser les registres du processeur comme bon vous semble, mais pour assurer l'interopérabilité avec d'autres logiciels, l'utilisation des registres est bien définie - c'est ce qu'on appelle la *convention d'appel*.
>
> Par exemple, les registres `x10` - `x11` sont utilisés pour les arguments et les valeurs de retour des fonctions. Pour une meilleure lisibilité, ils sont renommés `a0` - `a1` dans l'ABI. Consultez [la spécification](https://riscv.org/wp-content/uploads/2015/01/riscv-calling.pdf) pour plus de détails.

### Accès mémoire

Les registres sont très rapides mais limités en nombre. La plupart des données sont stockées en mémoire, et les programmes lisent/écrivent ces données avec les instructions `lw` (load word) et `sw` (store word) :

```asm
lw a0, (a1)  // Lit un mot (32 bits) à l'adresse contenue dans a1
             // et le stocke dans a0. En C : a0 = *a1;
```

```asm
sw a0, (a1)  // Stocke un mot de a0 à l'adresse contenue dans a1.
             // En C : *a1 = a0;
```

### Instructions de branchement

Les instructions de branchement modifient le flux de contrôle du programme. Elles sont utilisées pour implémenter `if`, `for` et `while` :

```asm
bnez a0, <label>   // Aller à <label> si a0 n'est pas nul
// Si a0 est nul, continuer ici

<label>:
// Si a0 n'est pas nul, continuer ici
```

`bnez` signifie "branch if not equal to zero". D'autres instructions courantes incluent `beq` (branch if equal) et `blt` (branch if less than). Elles sont similaires à `goto` en C, mais avec des conditions.

## Appels de fonction

Les instructions `jal` (jump and link) et `ret` (return) sont utilisées pour appeler des fonctions et en revenir :

```asm
    li  a0, 123      // Charge 123 dans le registre a0 (argument de la fonction)
    jal ra, <label>  // Saute à <label> et stocke l'adresse de retour
                     // dans le registre ra.

    // Après l'appel de la fonction, le programme continue ici...

// int func(int a) {
//   a += 1;
//   return a;
// }
<label>:
    addi a0, a0, 1    // Incrémente a0 (premier argument) de 1

    ret               // Retourne à l'adresse stockée dans ra.
                      // Le registre a0 contient la valeur de retour.
```

Les arguments des fonctions sont passés dans les registres `a0` - `a7`, et la valeur de retour est stockée dans le registre `a0`, conformément à la convention d'appel.

## Pile (Stack)

La pile est une zone mémoire de type Dernier Entré, Premier Sorti (LIFO) utilisée pour les appels de fonction et les variables locales. Elle croît vers les adresses inférieures, et le pointeur de pile `sp` indique le sommet de la pile.

Pour sauvegarder une valeur dans la pile, on décrémente le pointeur de pile et on stocke la valeur (opération *push*) :

```asm
    addi sp, sp, -4  // Déplace le pointeur de pile vers le bas de 4 octets
                     // (allocation sur la pile).

    sw   a0, (sp)    // Stocke a0 dans la pile
```

Pour charger une valeur depuis la pile, on la charge puis on incrémente le pointeur de pile (opération *pop*) :

```asm
    lw   a0, (sp)    // Charge a0 depuis la pile
    addi sp, sp, 4   // Déplace le pointeur de pile vers le haut de 4 octets
                     // (désallocation de la pile).
```

> [!ASTUCE]
>
> En C, les opérations sur la pile sont générées automatiquement par le compilateur, donc vous n'avez pas à les écrire manuellement.


### Modes du processeur

Le processeur dispose de plusieurs modes, chacun avec différents niveaux de privilège. En RISC-V, il y a trois modes :

| Mode   | Description |
| ------ | ----------------------------------- |
| M-mode | Mode où fonctionne OpenSBI (BIOS). |
| S-mode | Mode dans lequel fonctionne le noyau, aussi appelé "mode noyau". |
| U-mode | Mode dans lequel fonctionnent les applications, aussi appelé "mode utilisateur". |

### Instructions privilégiées

Certaines instructions sont dites privilégiées et ne peuvent pas être exécutées en mode utilisateur. Voici celles utilisées dans ce livre :

| Opcode et opérandes | Description |
| ------------------------ | -------------------------------------- |
| `csrr rd, csr`           | Lit depuis un CSR |
| `csrw csr, rs`           | Écrit dans un CSR |
| `csrrw rd, csr, rs`      | Lit et écrit simultanément un CSR |
| `sret`                   | Retourne du gestionnaire d'exceptions |
| `sfence.vma`             | Vide le cache de traduction d'adresses |

Les CSR (Control and Status Registers) stockent les paramètres du processeur. La liste complète est disponible dans la [spécification privilégiée RISC-V](https://riscv.org/specifications/privileged-isa/).


## Assembleur en ligne (Inline Assembly)

Dans les chapitres suivants, vous rencontrerez une syntaxe spécifique du langage C comme celle-ci :

```c
uint32_t value;
__asm__ __volatile__("csrr %0, sepc" : "=r"(value));
```

Ceci est de *l'assembleur en ligne*, une syntaxe permettant d'intégrer du code assembleur dans du code C. Bien que vous puissiez écrire l'assembleur dans un fichier séparé (extension `.S`), l'utilisation de l'assembleur en ligne est généralement préférée car :

- Vous pouvez utiliser des variables C dans l'assembleur et affecter les résultats de l'assembleur à des variables C.
- Vous pouvez laisser le compilateur gérer l'allocation des registres, évitant ainsi d'avoir à écrire manuellement la sauvegarde et la restauration des registres modifiés.

### Comment écrire de l'assembleur en ligne

L'assembleur en ligne est écrit selon le format suivant :

```c
__asm__ __volatile__("assembly" : opérandes de sortie : opérandes d'entrée : registres modifiés);
```

| Élément             | Description                                                      |
| ------------------- | ---------------------------------------------------------------- |
| `__asm__`           | Indique qu'il s'agit d'assembleur en ligne.                      |
| `__volatile__`      | Informe le compilateur de ne pas optimiser le code `"assembly"`. |
| `"assembly"`        | Code assembleur écrit sous forme de chaîne de caractères.        |
| Opérandes de sortie | Variables C stockant les résultats de l'assembleur.              |
| Opérandes d'entrée  | Expressions C (ex. `123`, `x`) utilisées dans l'assembleur.      |
| Registres modifiés  | Registres dont le contenu est altéré par l'assembleur.           |

Les opérandes de sortie et d'entrée sont séparés par des virgules et chaque opérande est écrit sous la forme `contrainte (expression C)`. Les contraintes définissent le type d'opérande, généralement `=r` (registre) pour la sortie et `r` pour l'entrée.

Les opérandes de sortie et d'entrée peuvent être référencés dans l'assembleur en utilisant `%0`, `%1`, `%2`, etc., en commençant par les opérandes de sortie.

### Exemples

```c
uint32_t value;
__asm__ __volatile__("csrr %0, sepc" : "=r"(value));
```

Cette instruction lit la valeur du CSR `sepc` en utilisant l'instruction `csrr` et l'affecte à la variable `value`. `%0` correspond à `value`.

```c
__asm__ __volatile__("csrw sscratch, %0" : : "r"(123));
```

Cette instruction écrit `123` dans le CSR `sscratch`, en utilisant l'instruction `csrw`. `%0` correspond au registre contenant `123` (`r` comme contrainte), et cela pourrait se traduire par :

```
li    a0, 123        // Charge 123 dans le registre a0
csrw  sscratch, a0   // Écrit la valeur de a0 dans le registre sscratch
```

Bien que seule l'instruction `csrw` soit écrite dans l'assembleur en ligne, l'instruction `li` est automatiquement insérée par le compilateur pour respecter la contrainte `"r"` (valeur dans un registre). C'est très pratique !

> [!ASTUCE]
>
> L'assembleur en ligne est une extension spécifique aux compilateurs et ne fait pas partie de la spécification du langage C. Vous pouvez consulter son utilisation détaillée dans la [documentation GCC](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html). Cependant, il peut être difficile à comprendre en raison de la syntaxe des contraintes qui varie selon l'architecture du processeur et des nombreuses fonctionnalités avancées.
>
> Pour les débutants, il est recommandé de chercher des exemples concrets. Par exemple, [HinaOS](https://github.com/nuta/microkernel-book/blob/52d66bd58cd95424f009e2df8bc1184f6ffd9395/kernel/riscv32/asm.h) et [xv6-riscv](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/riscv.h) sont de bonnes références.
