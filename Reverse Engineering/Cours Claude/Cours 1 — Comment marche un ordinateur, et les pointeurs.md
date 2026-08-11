
_Prérequis : bases Python, un peu de C. Objectif : avoir des fondations solides avant d'attaquer l'assembleur._

---

## 1. Les 3 acteurs principaux

|Composant|Rôle|Analogie|
|---|---|---|
|**CPU (processeur)**|Exécute les instructions une par une, très vite|Le cuisinier|
|**RAM (mémoire vive)**|Stocke temporairement code + données pendant l'exécution|Le plan de travail|
|**Stockage (disque/SSD)**|Stocke les fichiers de façon permanente|Le garde-manger|

Le CPU ne "comprend" que des **nombres binaires** (des 0 et des 1). Texte, image, code Python compilé, tout finit sous forme de bits en mémoire.

## 2. Le cycle Fetch-Decode-Execute

Le CPU répète en boucle, des milliards de fois par seconde :

1. **Fetch** (chercher) : il lit en RAM l'instruction pointée par le registre **EIP/RIP** ("où j'en suis dans le programme")
2. **Decode** (décoder) : il interprète cette suite de bits pour savoir quelle opération faire
3. **Execute** (exécuter) : il fait l'opération (addition, copie de donnée, saut ailleurs...)
4. Il incrémente EIP/RIP pour pointer l'instruction suivante, et recommence

**C'est tout.** Un programme entier n'est qu'une répétition géante de ce cycle.

## 3. Bits, octets, mots — les unités de base

|Unité|Taille|Exemple de valeurs possibles|
|---|---|---|
|**Bit**|1 chiffre binaire (0 ou 1)|0 ou 1|
|**Octet (byte)**|8 bits|0 à 255 (0x00 à 0xFF)|
|**Mot (word)**|16 bits (2 octets)|0 à 65535|
|**Double mot (dword)**|32 bits (4 octets)|~4,3 milliards de valeurs|
|**Quad mot (qword)**|64 bits (8 octets)|~18,4 trillions de valeurs|

En Python, `int` gère ça automatiquement pour toi. En C et en assembleur, **tu dois savoir combien d'octets occupe chaque variable** — c'est une différence fondamentale de mentalité à adopter.

### Hexadécimal : pourquoi c'est partout

Le binaire `01001000` est illisible pour un humain. L'hexadécimal (base 16, symboles 0-9 puis A-F) sert de raccourci : **chaque chiffre hexa = exactement 4 bits**.

```
Binaire   : 0100 1000
Hexa      :   4    8   →  0x48
```

Dans gdb/IDA/Ghidra, tout s'affiche en hexa (`0x401136`) — c'est juste une écriture compacte du binaire réel.

### Endianness (ordre des octets) — piège classique en reverse

Quand une valeur multi-octets est stockée en mémoire, l'ordre des octets compte :

|Type|Ordre|Qui l'utilise|
|---|---|---|
|**Little-endian**|Octet de poids **faible** en premier (adresse la plus basse)|x86, x86-64 (quasi tout ce que tu croiseras)|
|**Big-endian**|Octet de poids **fort** en premier|Certains réseaux (ordre "network byte order"), anciens systèmes|

Exemple : la valeur `0x12345678` stockée en mémoire à l'adresse 0x1000, en little-endian x86 :

```
Adresse :  0x1000  0x1001  0x1002  0x1003
Octet   :   0x78    0x56    0x34    0x12
```

**Mnémo** : "little" = le petit bout (le moins significatif) arrive en premier. C'est **contre-intuitif** la première fois — quand tu regardes un dump mémoire dans gdb et que tu vois `78 56 34 12`, sache que la vraie valeur logique est `0x12345678`, lue à l'envers.

---

## 4. Les adresses mémoire

La RAM est une immense rangée de cases, chaque case ayant un **numéro unique : son adresse**. Une adresse en 64 bits ressemble à `0x00007ffdf3a2b1c8`.

### L'espace d'adressage virtuel d'un programme

Chaque programme lancé voit sa **propre mémoire virtuelle**, organisée en zones distinctes :

```
Adresses hautes
┌─────────────────────┐
│   Stack (pile)       │  ← variables locales, adresses de retour — grandit vers le bas
│         ↓             │
│                       │
│         ↑             │
│   Heap (tas)          │  ← malloc/new — grandit vers le haut
├─────────────────────┤
│   .bss (data non init)│
│   .data (data init)   │
│   .text (le code)     │  ← tes instructions assembleur
└─────────────────────┘
Adresses basses
```

Tu reverras exactement ce découpage dans un fichier ELF/PE désassemblé — ce n'est pas juste une abstraction théorique, c'est littéralement comment le binaire est chargé en mémoire.

---

## 5. Les pointeurs — la notion centrale à maîtriser

### 5.1 En Python, tu ne les vois jamais (mais ils existent quand même)

```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)  # [1, 2, 3, 4] !
```

Ce comportement surprenant vient du fait que `a` et `b` sont en réalité deux **références** (des pointeurs cachés) vers le **même objet** en mémoire. Python te protège de ce détail, mais l'ordinateur, lui, manipule bien des adresses en coulisses.

### 5.2 En C, un pointeur est explicite

Un pointeur est **une variable qui contient une adresse mémoire**, pas une valeur directe.

```c
int x = 42;        // x est une variable normale, elle contient 42
int *p = &x;        // p est un pointeur, il contient l'ADRESSE de x
printf("%d\n", *p); // *p veut dire "va lire la valeur à l'adresse contenue dans p" -> 42
```

|Symbole|Nom|Rôle|
|---|---|---|
|`&x`|Opérateur "adresse de"|Donne l'adresse mémoire où est stockée `x`|
|`*p`|Opérateur "déréférencement"|Va lire/écrire la valeur stockée à l'adresse contenue dans `p`|
|`int *p`|Déclaration|`p` est un pointeur **vers** un `int` (il faut préciser le type pour savoir combien d'octets lire)|

**Analogie simple** : une variable normale est une boîte qui contient une valeur. Un pointeur est un post-it qui contient **l'adresse d'une autre boîte**. `*p` veut dire "va voir la boîte à l'adresse écrite sur le post-it, et regarde ce qu'il y a dedans".

### 5.3 Pourquoi ça compte énormément en reverse/sécu

En assembleur, il n'existe **pas de notion de "variable nommée"** comme en C ou en Python — tout est adresse mémoire ou registre. Quand tu vois en C :

```c
int tab[5];
tab[2] = 10;
```

Ça se traduit en assembleur par un calcul d'adresse : _"adresse de base de `tab`, plus 2 fois la taille d'un int (4 octets), écris 10 à cette adresse"_. C'est exactement ce que fait l'instruction `LEA` (Load Effective Address) que tu croiseras énormément en désassemblage.

**C'est aussi la base de tous les bugs mémoire** :

- Un **pointeur qui pointe hors des limites autorisées** = accès mémoire invalide (souvent un crash, ou pire, une exploitation possible)
- Un **buffer trop petit rempli avec trop de données** = les octets en trop écrasent la mémoire juste après = **buffer overflow** (on y reviendra dans le cours dédié)
- Un **pointeur non initialisé ou libéré puis réutilisé** = comportement indéfini (`use-after-free`, très exploité en sécurité)

### 5.4 Arithmétique de pointeurs

```c
int tab[5] = {10, 20, 30, 40, 50};
int *p = tab;      // p pointe sur tab[0]
p = p + 1;          // p pointe maintenant sur tab[1] !
```

`p + 1` n'ajoute **pas** 1 à l'adresse — il ajoute `1 × sizeof(int)` = 4 octets, parce que le compilateur sait que `p` pointe vers des `int`. C'est le compilateur qui fait ce calcul pour toi en C ; en assembleur, **ce calcul est explicite** et visible dans chaque instruction d'adressage.

---

## 6. Du code source au binaire — la chaîne de compilation

```
Code source (.c)
      │  compilateur (gcc, clang...)
      ▼
Assembleur (.s)      ← MOV/ADD/JMP, lisible par un humain
      │  assembleur (as)
      ▼
Code objet (.o)      ← déjà en binaire, pas encore lié
      │  éditeur de liens (linker)
      ▼
Exécutable final (ELF sous Linux / PE sous Windows)
```

L'assembleur **n'est pas un langage arbitraire** — c'est la traduction lisible du binaire que le CPU exécute réellement. En reverse engineering, tu fais le chemin inverse : tu pars du binaire et tu remontes vers de l'assembleur lisible via un désassembleur.

---

## 7. Ce qu'il faut retenir avant la suite

1. Tout est stocké en mémoire sous forme d'octets à des adresses précises.
2. Little-endian = l'octet de poids faible est stocké en premier (x86).
3. Un pointeur = une variable qui contient une adresse, pas une valeur directe.
4. `&` = donne une adresse ; `*` = va lire à cette adresse.
5. L'assembleur n'a que des adresses et des registres — pas de "variables nommées" comme en C/Python. Comprendre les pointeurs, c'est déjà comprendre la moitié de ce que fait le code assembleur.

**Prochain cours** : toutes les instructions assembleur, catégorie par catégorie.