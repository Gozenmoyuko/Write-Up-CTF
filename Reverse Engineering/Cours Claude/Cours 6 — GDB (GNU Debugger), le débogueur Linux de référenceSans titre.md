## 1. Qu'est-ce que GDB fait concrètement

GDB te permet d'exécuter un programme **pas à pas**, en observant en temps réel :

- Les valeurs des registres (EAX, ESP, EIP...)
- Le contenu de la mémoire
- La pile
- Le code assembleur exécuté

C'est l'outil pour l'**analyse dynamique** (par opposition à Ghidra/IDA qui font surtout de l'**analyse statique**, sans exécuter le programme).

---

## 2. Installation et configuration de base

```bash
sudo apt install gdb
```

Par défaut, GDB affiche l'assembleur en syntaxe **AT&T**. Pour passer en syntaxe **Intel** (recommandé, plus lisible pour débuter) :

```
(gdb) set disassembly-flavor intel
```

Pour rendre ça permanent, ajoute cette ligne dans `~/.gdbinit`.

### Extensions indispensables : pwndbg / GEF

GDB "nu" est assez austère. Pour du reverse/pwn, installe une extension :

- **pwndbg** : `https://github.com/pwndbg/pwndbg` — très populaire en CTF, affiche automatiquement registres, pile, désassemblage à chaque étape
- **GEF (GDB Enhanced Features)** : `https://github.com/hugsy/gef` — alternative similaire, très utilisée aussi

Les deux transforment GDB en interface bien plus lisible, avec des commandes en plus (comme `cyclic` pour générer des patterns de fuzzing).

---

## 3. Commandes essentielles

### Lancer et contrôler l'exécution

|Commande|Rôle|
|---|---|
|`gdb ./programme`|Charge le binaire dans GDB|
|`run` (ou `r`)|Lance l'exécution|
|`run arg1 arg2`|Lance avec des arguments|
|`start`|Lance et s'arrête automatiquement au début de `main`|
|`continue` (ou `c`)|Continue l'exécution jusqu'au prochain breakpoint|
|`next` (ou `n`)|Exécute la ligne/instruction suivante **sans entrer** dans les sous-fonctions|
|`step` (ou `s`)|Exécute la ligne/instruction suivante, **entre** dans les sous-fonctions|
|`nexti` / `stepi`|Équivalent de next/step mais instruction par instruction (asm) au lieu de ligne par ligne|
|`finish`|Termine la fonction courante et revient à l'appelant|
|`kill`|Arrête le programme en cours d'exécution|

### Points d'arrêt (breakpoints)

|Commande|Rôle|
|---|---|
|`break main` (ou `b main`)|Pose un point d'arrêt sur la fonction `main`|
|`break *0x401136`|Pose un point d'arrêt sur une adresse précise|
|`break fichier.c:42`|Point d'arrêt sur une ligne de code source (si le binaire a des symboles de debug)|
|`info breakpoints`|Liste tous les breakpoints actifs|
|`delete 1`|Supprime le breakpoint n°1|
|`watch variable`|Point d'arrêt qui se déclenche quand une variable **change de valeur** (watchpoint)|

### Observer l'état du programme

|Commande|Rôle|
|---|---|
|`info registers` (ou `i r`)|Affiche tous les registres généraux|
|`info registers eax`|Affiche un seul registre|
|`print $eax` (ou `p $eax`)|Affiche la valeur d'un registre|
|`print/x $eax`|Affiche en hexadécimal|
|`x/4xb $esp`|Examine 4 octets (b=byte) en hexa (x) à l'adresse ESP|
|`x/10i $eip`|Affiche les 10 prochaines instructions (i=instruction) depuis EIP|
|`x/s $eax`|Affiche la chaîne de caractères (s=string) pointée par EAX|
|`disas` (ou `disassemble`)|Désassemble la fonction courante|
|`disas main`|Désassemble une fonction précise|
|`backtrace` (ou `bt`)|Affiche la pile d'appels (quelles fonctions ont appelé quoi)|
|`info frame`|Détails sur la stack frame courante|

### La commande `x` en détail (eXamine memory) — très utilisée

Syntaxe : `x/NFU adresse`

|Lettre|Signifie|Valeurs possibles|
|---|---|---|
|N|Nombre d'unités à afficher|un chiffre, ex : `10`|
|F|Format|`x`=hexa, `d`=décimal, `s`=string, `i`=instruction, `c`=char|
|U|Unité (taille)|`b`=byte(1), `h`=halfword(2), `w`=word(4), `g`=giant(8)|

Exemple : `x/20xb $esp` = affiche les 20 prochains octets depuis ESP, en hexadécimal.

---

## 4. Workflow type pour analyser un binaire

```bash
gdb ./vulnerable
```

```
(gdb) set disassembly-flavor intel
(gdb) break main
(gdb) run
(gdb) disas               # voir le code assembleur de main
(gdb) next                 # avancer instruction par instruction
(gdb) info registers        # vérifier l'état des registres
(gdb) x/20xb $esp          # inspecter la pile
(gdb) continue
```

### Pour identifier un offset de buffer overflow (pattern cyclique)

Avec pwndbg/GEF installé :

```
(gdb) cyclic 200                     # génère un pattern unique de 200 octets
(gdb) run                             # lance avec ce pattern en entrée
# le programme crashe, EIP contient une partie du pattern
(gdb) cyclic -l 0x6161616c            # trouve l'offset exact correspondant à cette valeur
```

Cette technique permet de savoir **exactement combien d'octets** il faut avant d'atteindre l'adresse de retour — la première étape de tout exploit de buffer overflow (voir Cours 10).

---

## 5. Ce qu'il faut retenir

1. GDB = analyse **dynamique**, en exécutant réellement le programme pas à pas.
2. `break`/`run`/`next`/`step`/`continue` pour contrôler l'exécution.
3. `info registers` et `x/...` pour observer registres et mémoire.
4. Toujours installer pwndbg ou GEF — GDB nu est fonctionnel mais peu lisible pour débuter.
5. `set disassembly-flavor intel` dès le début pour rester cohérent avec les autres outils (IDA/Ghidra).

**Prochain cours** : IDA — l'outil de référence historique pour l'analyse statique.