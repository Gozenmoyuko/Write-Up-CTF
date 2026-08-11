
Tout d'abord avant d'attaquer il faut savoir que le cours que vous allez commencer à lire n'est pas fais de mes mains. J'en ferais une de mes mains par la suite que je ferais avec des explications plus claire que j'ai compris. Il faut savoir que j'ai donc générer via Claude et via ce prompt : 

"est-ce qu'avant la fiche tu peux faire quelques explication basique du langage Assembleur et la différence et un petit cours sur les ordinateurs vraiment pour avoir de bonne base pour commencer à attaquer le reverse engineering."

Pour vous mettres dans le contexte je demander surtout un cours sur les registres et par la suite j'ai voulu quand même avant de mettre le cours sur les registres de mettre des bases pour ceux qui ne l'ont pas. 

Donc voici le cours qui est plutôt pas mal en réalité.




# Bases avant le Reverse Engineering — Ordinateur, Assembleur, Architecture

## 1. Comment fonctionne un ordinateur (vue minimale mais suffisante)

### 1.1 Les 3 acteurs principaux

|Composant|Rôle|Analogie|
|---|---|---|
|**CPU (processeur)**|Exécute les instructions une par une, très vite|Le cuisinier|
|**RAM (mémoire)**|Stocke temporairement code + données pendant l'exécution|Le plan de travail|
|**Stockage (disque)**|Stocke les fichiers de façon permanente|Le garde-manger|

Le CPU ne "comprend" que des **nombres binaires** (des 0 et des 1). Tout ce que tu vois — texte, image, code — finit sous forme de suites de bits en mémoire.

### 1.2 Le cycle Fetch-Decode-Execute (le cœur de tout)

Le CPU répète en boucle, des milliards de fois par seconde :

1. **Fetch** (chercher) : il va lire en RAM l'instruction pointée par **EIP/RIP** (le registre qu'on a vu dans la fiche — "où j'en suis dans le programme")
2. **Decode** (décoder) : il interprète cette suite de bits pour savoir quelle opération faire
3. **Execute** (exécuter) : il fait l'opération (addition, copie de donnée, saut ailleurs dans le code...)
4. Il incrémente EIP/RIP pour pointer l'instruction suivante, et recommence.

**C'est tout.** Un programme entier, aussi complexe soit-il, n'est qu'une répétition géante de ce cycle. Comprendre ça, c'est comprendre 80% de ce qu'il se passe quand tu fais du reverse.

### 1.3 Registres vs RAM — pourquoi cette distinction compte

||Registres (EAX, EBX...)|RAM|
|---|---|---|
|Vitesse|Ultra rapide (dans le CPU même)|Beaucoup plus lente|
|Capacité|Quelques dizaines d'octets seulement|Des Go|
|Usage|Calculs immédiats, valeurs "en cours d'utilisation"|Stockage de toutes les variables, tableaux, la pile|

**Mnémo** : les registres sont la mémoire "de poche" du CPU — il y va chercher/stocker ce dont il a besoin _maintenant_, puis renvoie le résultat en RAM s'il faut le garder.

### 1.4 Binaire, Hexadécimal : pourquoi c'est partout en reverse

Le CPU raisonne en binaire, mais un humain ne lit pas facilement `01001000`. On utilise donc l'**hexadécimal** (base 16, symboles 0-9 puis A-F) comme raccourci : chaque chiffre hexa = exactement 4 bits.

```
Binaire   : 0100 1000
Hexa      :   4    8
```

C'est pour ça que dans un désassembleur/débogueur (gdb, IDA, Ghidra) tu verras tout en hexa (`0x401136`, `0x90`...) — c'est juste une écriture plus lisible du binaire réel.

---

## 2. Du code source au binaire — la chaîne de compilation

Quand tu écris du C, voici ce qu'il se passe avant que ça devienne un fichier exécutable :

```
Code source (.c)
      │  compilateur (gcc, clang...)
      ▼
Assembleur (.s)      ← c'est ICI qu'on est, avec MOV/ADD/JMP...
      │  assembleur (as)
      ▼
Code objet (.o)      ← déjà en binaire, mais pas encore lié
      │  éditeur de liens (linker)
      ▼
Exécutable final (ELF sous Linux / PE sous Windows)
```

**Point essentiel** : l'assembleur n'est **pas** un langage à part entière avec une syntaxe arbitraire — c'est la **traduction lisible par un humain** du binaire que le CPU exécute réellement. Chaque ligne d'assembleur correspond (presque) à une instruction machine unique.

En reverse engineering, tu fais **le chemin inverse** : tu pars du binaire (0 et 1) et tu remontes vers de l'assembleur lisible (via un désassembleur comme objdump/IDA/Ghidra/radare2), pour comprendre ce que fait le programme sans avoir le code source.

---

## 3. Assembleur : les familles et les syntaxes

### 3.1 Différentes architectures = différents "dialectes"

|Architecture|Où on la trouve|Style|
|---|---|---|
|**x86 / x86-64**|PC, laptops, la plupart des serveurs|CISC (instructions riches, complexes)|
|**ARM / ARM64**|Smartphones, Mac M1/M2/M3, IoT|RISC (instructions simples, uniformes)|
|**MIPS**|Anciens routeurs, certains systèmes embarqués|RISC|

**CISC vs RISC en une phrase** : CISC (x86) a beaucoup d'instructions complexes qui font plusieurs choses à la fois ; RISC (ARM) a peu d'instructions simples, mais s'exécute souvent de façon plus prévisible. Pour du reverse sur PC/Windows/Linux classique, tu seras à 95% sur x86/x86-64 — c'est pour ça qu'on s'est concentrés dessus dans la fiche.

### 3.2 Deux syntaxes pour la même chose : Intel vs AT&T

Même instruction, deux écritures possibles :

|Syntaxe|Exemple|Ordre|
|---|---|---|
|**Intel** (Windows, IDA, Ghidra par défaut)|`mov eax, ebx`|destination, source|
|**AT&T** (Linux/GDB par défaut, GCC)|`mov %ebx, %eax`|source, destination|

**Mnémo** : en Intel, tu lis de gauche à droite comme une affectation `eax = ebx` (destination d'abord). En AT&T, c'est l'inverse, et les registres ont un `%` devant, les valeurs immédiates un `$`.

Sous Kali/gdb tu peux forcer la syntaxe Intel avec `set disassembly-flavor intel` — beaucoup plus lisible quand on démarre.

---

## 4. Les catégories d'instructions à connaître avant tout le reste

Tu n'as pas besoin de mémoriser les ~1000 instructions x86 qui existent. **90% du reverse de base repose sur une dizaine d'instructions :**

|Catégorie|Instructions clés|Rôle|
|---|---|---|
|**Déplacement de données**|`MOV`, `LEA`, `PUSH`, `POP`|Copier une valeur d'un endroit à un autre|
|**Arithmétique**|`ADD`, `SUB`, `MUL`, `DIV`, `INC`, `DEC`|Calculs|
|**Logique**|`AND`, `OR`, `XOR`, `NOT`|Opérations bit à bit|
|**Comparaison**|`CMP`, `TEST`|Compare deux valeurs et met à jour les FLAGS (vu dans la fiche)|
|**Saut inconditionnel**|`JMP`|"Va à telle adresse", sans condition|
|**Saut conditionnel**|`JE`, `JNE`, `JG`, `JL`, `JZ`...|"Va à telle adresse SI le flag correspondant est activé"|
|**Fonctions**|`CALL`, `RET`|Appeler une fonction / y revenir|

**Mnémo pour les sauts conditionnels** : ils se lisent comme des questions posées aux FLAGS.

- `JE` = **J**ump if **E**qual (ZF=1)
- `JNE` = **J**ump if **N**ot **E**qual (ZF=0)
- `JG` = **J**ump if **G**reater
- `JL` = **J**ump if **L**ess
- `JZ` / `JNZ` = **J**ump if **Z**ero / **N**ot **Z**ero (même chose que JE/JNE en pratique)

C'est exactement ce que fait un `if () {}` en C une fois compilé : `CMP` compare, puis un `Jxx` décide où sauter selon le résultat.

---

## 5. Ce que tu dois retenir pour attaquer le reverse

1. **Un programme = une suite d'instructions en mémoire, exécutées une par une via EIP/RIP.**
2. **Les registres sont l'espace de travail immédiat du CPU** — c'est pour ça qu'on les étudie en premier (voir la fiche associée).
3. **CMP + Jxx = un `if` du code source** une fois compilé — repérer ce duo te permet de reconstruire la logique du programme.
4. **PUSH/POP/CALL/RET gèrent la pile** — indispensable pour comprendre les appels de fonction et les buffer overflows.
5. **La syntaxe (Intel vs AT&T) ne change rien au sens**, juste l'ordre d'écriture — ne te laisse pas perturber en changeant d'outil.

### Outils pour t'entraîner concrètement

|Outil|Usage|
|---|---|
|`gdb` + `pwndbg`/`gef`|Débogage pas à pas, voir les registres en live|
|`objdump -d`|Désassembler rapidement un binaire en ligne de commande|
|**Ghidra** (gratuit, NSA)|Décompilation + reverse statique avec interface graphique|
|**radare2 / Cutter**|Alternative open-source à Ghidra/IDA|
|**IDA Free**|Référence historique du reverse, version gratuite limitée|

**Prochaine étape logique** : compiler un petit `.c` avec une boucle et un `if`, le désassembler avec `objdump -d -M intel`, et repérer toi-même le CMP/Jxx correspondant à ton `if`. C'est l'exercice le plus formateur pour faire le lien code source ↔ assembleur.