

_Note honnête avant de commencer : le jeu d'instructions x86 complet compte plusieurs centaines d'opcodes (en comptant toutes les variantes SIMD/AVX). Lister littéralement chaque encodage binaire n'aurait aucune utilité pédagogique. Ce cours couvre en revanche **toutes les familles/catégories d'instructions qui existent**, avec un luxe de détails sur celles que tu croiseras réellement en reverse engineering — ce qui représente en pratique tout ce dont tu as besoin._

---

## 1. Comment lire une instruction assembleur

Une instruction a cette forme générale (syntaxe Intel, celle qu'on utilise ici) :

```
MNEMONIQUE  DESTINATION, SOURCE
```

Exemple : `MOV EAX, EBX` → "copie la valeur de EBX dans EAX" (destination à gauche, comme une affectation `EAX = EBX`).

### Les modes d'adressage (comment on désigne un opérande)

|Mode|Exemple|Signification|
|---|---|---|
|**Immédiat**|`MOV EAX, 5`|La valeur littérale 5|
|**Registre**|`MOV EAX, EBX`|Le contenu du registre EBX|
|**Direct (mémoire)**|`MOV EAX, [0x4010]`|La valeur stockée à l'adresse mémoire 0x4010|
|**Indirect (registre)**|`MOV EAX, [EBX]`|La valeur à l'adresse **contenue dans** EBX (comme `*p` en C !)|
|**Indexé avec base**|`MOV EAX, [EBX+ECX*4]`|Base + index × taille — exactement `tab[i]` en C|
|**Base + déplacement**|`MOV EAX, [EBP-4]`|Une variable locale sur la pile|

**Lien direct avec le cours précédent** : `[EBX]` en assembleur = `*p` en C (déréférencement). `[EBX+ECX*4]` = `tab[i]` avec des `int` (4 octets). Les crochets `[...]` signifient toujours "va lire en mémoire à cette adresse calculée".

---

## 2. Catégorie 1 — Transfert de données

| Instruction  | Nom complet                        | Rôle détaillé                                                                                                                                                                                                                               |
| ------------ | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MOV`        | **Mov**e                           | Copie une valeur (registre→registre, mémoire→registre, registre→mémoire, immédiat→registre). L'instruction la plus utilisée, sans exception.                                                                                                |
| `LEA`        | **L**oad **E**ffective **A**ddress | Calcule une adresse (via les crochets) mais **ne va PAS lire en mémoire** — il donne juste le résultat du calcul. `LEA EAX, [EBX+4]` met `EBX+4` dans EAX, sans toucher à la mémoire. Souvent détourné pour faire de l'arithmétique rapide. |
| `MOVZX`      | **Mov**e with **Z**ero e**X**tend  | Copie une petite valeur (8/16 bits) vers un registre plus grand, en remplissant les bits du haut avec des 0 (pour un nombre non signé)                                                                                                      |
| `MOVSX`      | **Mov**e with **S**ign e**X**tend  | Pareil mais remplit avec le bit de signe (pour un nombre signé négatif/positif)                                                                                                                                                             |
| `PUSH`       | **Push** (empiler)                 | Décrémente ESP/RSP puis écrit la valeur au sommet de la pile                                                                                                                                                                                |
| `POP`        | **Pop** (dépiler)                  | Lit la valeur au sommet de la pile puis incrémente ESP/RSP                                                                                                                                                                                  |
| `XCHG`       | E**xch**an**g**e                   | Échange le contenu de deux opérandes                                                                                                                                                                                                        |
| `IN` / `OUT` | **In**put / **Out**put             | Lecture/écriture sur un port matériel (bas niveau, rare en reverse applicatif)                                                                                                                                                              |
| `CMOVcc`     | **C**onditional **Mov**e           | Copie seulement si une condition (flag) est vraie — évite un saut conditionnel                                                                                                                                                              |

---

## 3. Catégorie 2 — Arithmétique

|Instruction|Nom complet|Rôle|
|---|---|---|
|`ADD`|**Add**ition|Addition simple : `dest = dest + source`|
|`ADC`|**Ad**d with **C**arry|Addition en tenant compte de la retenue précédente (pour additionner des nombres sur plusieurs registres)|
|`SUB`|**Sub**traction|Soustraction : `dest = dest - source`|
|`SBB`|**S**u**b**tract with **B**orrow|Soustraction avec emprunt|
|`INC`|**Inc**rement|Ajoute 1|
|`DEC`|**Dec**rement|Soustrait 1|
|`NEG`|**Neg**ate|Change le signe (complément à deux)|
|`MUL`|**Mul**tiply (non signé)|Multiplication non signée|
|`IMUL`|**I**nteger **Mul**tiply (signé)|Multiplication signée|
|`DIV`|**Div**ide (non signé)|Division non signée|
|`IDIV`|**I**nteger **Div**ide (signé)|Division signée|
|`CMP`|**Comp**are|Fait `dest - source` **sans stocker le résultat**, juste pour mettre à jour les FLAGS. Base de tous les `if`|

---

## 4. Catégorie 3 — Logique et manipulation de bits

|Instruction|Nom complet|Rôle|
|---|---|---|
|`AND`|**And** logique|ET bit à bit|
|`OR`|**Or** logique|OU bit à bit|
|`XOR`|**X**clusive **OR**|OU exclusif. Astuce classique : `XOR EAX, EAX` met EAX à 0, plus rapide que `MOV EAX, 0`|
|`NOT`|**Not** logique|Inverse tous les bits (complément à un)|
|`TEST`|**Test**|Fait un `AND` **sans stocker le résultat**, juste pour mettre à jour les FLAGS (souvent `TEST EAX, EAX` pour tester si EAX == 0)|
|`SHL` / `SAL`|**Sh**ift **L**eft / **S**hift **A**rithmetic **L**eft|Décalage de bits vers la gauche (≈ multiplier par 2^n)|
|`SHR`|**Sh**ift **R**ight|Décalage vers la droite non signé (≈ diviser par 2^n)|
|`SAR`|**S**hift **A**rithmetic **R**ight|Décalage vers la droite signé (conserve le bit de signe)|
|`ROL` / `ROR`|**Ro**tate **L**eft / **R**ight|Rotation circulaire des bits|
|`RCL` / `RCR`|**R**otate through **C**arry Left/Right|Rotation en passant par le flag Carry|
|`BT` / `BTS` / `BTR`|**B**it **T**est / **T**est and **S**et / **T**est and **R**eset|Teste et/ou modifie un bit précis|

---

## 5. Catégorie 4 — Contrôle de flux (sauts et fonctions)

### Sauts inconditionnels et appels

|Instruction|Nom complet|Rôle|
|---|---|---|
|`JMP`|**J**u**mp**|Saute directement à une adresse, sans condition|
|`CALL`|**Call**|Empile l'adresse de retour, puis saute vers la fonction|
|`RET`|**Ret**urn|Dépile l'adresse de retour et y saute — retour de fonction|
|`LOOP`|**Loop**|Décrémente ECX/RCX puis saute si ECX ≠ 0 (boucle automatique)|
|`INT`|**Int**errupt|Déclenche une interruption logicielle (appels système historiques, ex : `INT 0x80` sous Linux 32 bits)|
|`SYSCALL` / `SYSENTER`|—|Appel système rapide (remplace INT sur les CPU modernes)|
|`IRET`|**I**nterrupt **Ret**urn|Retour d'une routine d'interruption|
|`HLT`|**H**a**lt**|Arrête le CPU jusqu'à la prochaine interruption|
|`NOP`|**N**o **Op**eration|Ne fait rien (souvent `0x90`, utilisé en padding ou en exploitation — "NOP sled")|

### Sauts conditionnels (Jcc) — la famille complète

Tous testent les FLAGS mis à jour par un `CMP` ou `TEST` précédent.

|Instruction|Condition testée|Utilisé après CMP pour...|
|---|---|---|
|`JE` / `JZ`|ZF=1|égal / zéro|
|`JNE` / `JNZ`|ZF=0|différent / non-zéro|
|`JG` / `JNLE`|ZF=0 et SF=OF|supérieur (signé)|
|`JGE` / `JNL`|SF=OF|supérieur ou égal (signé)|
|`JL` / `JNGE`|SF≠OF|inférieur (signé)|
|`JLE` / `JNG`|ZF=1 ou SF≠OF|inférieur ou égal (signé)|
|`JA` / `JNBE`|CF=0 et ZF=0|supérieur (non signé)|
|`JAE` / `JNB`|CF=0|supérieur ou égal (non signé)|
|`JB` / `JNAE`|CF=1|inférieur (non signé)|
|`JBE` / `JNA`|CF=1 ou ZF=1|inférieur ou égal (non signé)|
|`JC`|CF=1|retenue (carry) présente|
|`JNC`|CF=0|pas de retenue|
|`JO` / `JNO`|OF=1 / OF=0|dépassement / pas de dépassement|
|`JS` / `JNS`|SF=1 / SF=0|résultat négatif / positif|

**Piège classique pour débutant** : `JG`/`JL` (signé) et `JA`/`JB` (non signé) testent la **même comparaison arithmétique**, mais donnent des résultats différents sur des valeurs comme `-1` — parce que `-1` en non-signé vaut `0xFFFFFFFF` (un très grand nombre). Toujours vérifier si les données manipulées sont censées être signées ou non.

---

## 6. Catégorie 5 — Instructions sur les chaînes/tableaux

|Instruction|Nom complet|Rôle|
|---|---|---|
|`MOVS`|**Mov**e **S**tring|Copie un élément de [ESI] vers [EDI], puis avance les deux pointeurs|
|`CMPS`|**C**o**mp**are **S**tring|Compare [ESI] et [EDI]|
|`SCAS`|**S**can **S**tring|Compare EAX/AL avec [EDI] (recherche d'un caractère, ex: `strlen`)|
|`LODS`|**Lo**a**d** **S**tring|Charge [ESI] dans EAX/AL|
|`STOS`|**Sto**re **S**tring|Écrit EAX/AL dans [EDI]|
|`REP`|**Rep**eat (préfixe)|Répète l'instruction suivante ECX fois — combiné à STOS ça fait un `memset`, à MOVS un `memcpy`|
|`REPE`/`REPZ`, `REPNE`/`REPNZ`|Repeat while (Not) Equal/Zero|Répète tant que la condition est vraie (utilisé avec CMPS/SCAS, ex : `strcmp`, `strlen`)|

---

## 7. Catégorie 6 — Pile (approfondissement, voir aussi le Cours 4 dédié)

|Instruction|Rôle|
|---|---|
|`PUSH` / `POP`|Empiler/dépiler une valeur|
|`PUSHF` / `POPF`|Empiler/dépiler le registre EFLAGS|
|`PUSHA` / `POPA`|Empiler/dépiler tous les registres généraux d'un coup (obsolète en 64 bits)|
|`ENTER`|Crée une stack frame en une instruction (rarement généré par les compilateurs modernes)|
|`LEAVE`|Équivalent de `MOV ESP, EBP` + `POP EBP` — nettoie la stack frame avant un RET|

---

## 8. Catégorie 7 — Système et bas niveau

|Instruction|Nom complet|Rôle|
|---|---|---|
|`CPUID`|**CPU** **ID**entification|Renvoie des infos sur le processeur (souvent utilisé en anti-debug/anti-VM pour détecter un environnement d'analyse)|
|`RDTSC`|**R**ea**d** **T**ime **S**tamp **C**ounter|Lit un compteur de cycles CPU (aussi utilisé en anti-debug, pour détecter un ralentissement suspect)|
|`LGDT`, `LIDT`, `LTR`|—|Chargement de structures système bas niveau (mode noyau uniquement)|
|`CLI` / `STI`|**Cl**ear/**S**et **I**nterrupt flag|Active/désactive les interruptions (mode noyau)|
|`RDMSR` / `WRMSR`|Read/Write Model-Specific Register|Accès aux registres spéciaux du CPU (mode noyau)|

Ces instructions sont **privilégiées** (mode noyau uniquement) — tu les croiseras surtout en analyse de rootkits/drivers, pas en reverse applicatif classique.

---

## 9. Catégorie 8 — Virgule flottante (x87) et SIMD (aperçu)

Le x86 gère les nombres à virgule flottante et le calcul parallèle via des unités séparées :

|Famille|Exemples d'instructions|Rôle|
|---|---|---|
|**x87 (FPU historique)**|`FLD`, `FST`, `FADD`, `FSUB`, `FMUL`, `FDIV`, `FCOMP`|Calcul flottant à l'ancienne, pile de 8 registres dédiés (ST0-ST7)|
|**MMX**|`PADDB`, `PSUBW`, `PAND`...|Calcul entier parallèle (multimédia, obsolète aujourd'hui)|
|**SSE/SSE2/SSE3/SSE4**|`MOVAPS`, `ADDPS`, `MULPS`, `CVTSI2SS`...|Calcul flottant vectoriel moderne (registres XMM0-XMM15), **c'est ce qu'utilise le compilateur aujourd'hui pour `float`/`double`**|
|**AVX/AVX2/AVX-512**|`VADDPS`, `VMULPD`...|Extension des registres SSE (YMM/ZMM, jusqu'à 512 bits), calcul massivement parallèle|

**Ce qu'il faut retenir sans apprendre chaque instruction par cœur** : si tu vois un préfixe `V` (`VMOVAPS`) ou un registre `XMM`/`YMM`, c'est du calcul flottant ou vectorisé — le compilateur a transformé une simple opération sur des `float`/`double` en instructions SIMD pour aller plus vite. Ça complique la lecture mais ne change pas la logique du programme.

---

## 10. FICHE FINALE — Les instructions vraiment utiles au quotidien

_C'est cette liste-là qu'il faut connaître par cœur en premier — elle couvre 90% de ce que tu liras en reverse/CTF._

|Instruction|Rôle en une phrase|
|---|---|
|**MOV**|Copie une valeur|
|**LEA**|Calcule une adresse sans y accéder|
|**PUSH / POP**|Empile/dépile sur la pile|
|**ADD / SUB**|Addition / soustraction|
|**CMP**|Compare deux valeurs (met à jour les flags)|
|**TEST**|Teste des bits, souvent `TEST EAX,EAX` pour tester si nul|
|**JMP**|Saut inconditionnel|
|**JE/JNE/JZ/JNZ**|Sauts conditionnels les plus fréquents (égal/différent)|
|**JG/JL/JGE/JLE**|Sauts conditionnels signés (comparaisons `>`, `<`)|
|**CALL / RET**|Appel et retour de fonction|
|**XOR**|Souvent utilisé pour mettre un registre à 0 (`XOR EAX,EAX`)|
|**NOP**|Ne fait rien (0x90)|
|**AND / OR**|Logique bit à bit|
|**INC / DEC**|+1 / -1|
|**REP MOVS / REP STOS**|Équivalents assembleur de `memcpy`/`memset`|

**Astuce d'apprentissage** : prends un petit programme C, compile-le (`gcc -O0 -o prog prog.c`), désassemble-le (`objdump -d -M intel prog`), et repère ces instructions une par une dans le résultat. C'est bien plus efficace que d'apprendre la liste par cœur dans le vide.

**Prochain cours** : les registres en détail (fiche dédiée déjà réalisée), puis la pile en profondeur.