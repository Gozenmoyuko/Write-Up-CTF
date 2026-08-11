## 1. Pourquoi ces formats existent

Après compilation et édition de liens, un programme n'est pas juste "du binaire brut jeté en vrac" — il suit un **format de conteneur structuré** qui dit à l'OS comment le charger en mémoire et l'exécuter (voir chaîne de compilation, Cours 1).

- **ELF** (_Executable and Linkable Format_) → Linux et la plupart des Unix
- **PE** (_Portable Executable_) → Windows (.exe, .dll)

Même rôle des deux côtés : un **plan de montage** pour l'OS, mais structure interne différente.

---

## 2. Anatomie comparée

|Élément|Rôle|Nom ELF|Nom PE|
|---|---|---|---|
|En-tête (header)|Identifie le format, l'architecture, le point d'entrée|ELF Header|PE Header (DOS + NT Headers)|
|Point d'entrée|Adresse où EIP/RIP démarre au lancement|`e_entry`|`AddressOfEntryPoint`|
|Section code|Instructions assembleur exécutables|`.text`|`.text`|
|Données initialisées|Variables globales avec valeur de départ|`.data`|`.data`|
|Données non-initialisées|Variables globales déclarées mais vides au départ|`.bss`|`.bss`|
|Fonctions importées|Fonctions utilisées venant de bibliothèques externes|Table de relocation / `.dynsym`|**Import Table (IAT)**|
|Fonctions exportées|Fonctions proposées par ce binaire (surtout .dll/.so)|`.dynsym` (symboles exportés)|**Export Table**|

**Mnémo** : la première chose que fait Ghidra/IDA en ouvrant un binaire, c'est lire ce header pour savoir "où commence le code" et "quelles sections existent" — exactement ce que fait l'OS au lancement.

---

## 3. Structure détaillée d'un ELF

```
┌───────────────────┐
│  ELF Header           │  ← magic number 0x7F 'E' 'L' 'F', architecture, point d'entrée
├───────────────────┤
│  Program Headers      │  ← comment charger le fichier en mémoire (segments)
├───────────────────┤
│  .text                 │  ← code exécutable
│  .rodata               │  ← données en lecture seule (chaînes constantes...)
│  .data                 │  ← variables globales initialisées
│  .bss                  │  ← variables globales non initialisées
│  .dynsym / .dynstr     │  ← symboles dynamiques et leurs noms
│  .plt / .got           │  ← voir section dédiée ci-dessous
├───────────────────┤
│  Section Headers      │  ← table décrivant chaque section (nom, taille, permissions)
└───────────────────┘
```

Commande utile : `readelf -h ./binaire` (header), `readelf -S ./binaire` (sections), `file ./binaire` (identification rapide).

---

## 4. Structure détaillée d'un PE

```
┌───────────────────┐
│  DOS Header            │  ← héritage historique ("MZ" magic number, message "This program cannot be run in DOS mode")
├───────────────────┤
│  NT Headers            │  ← vrai header PE : signature "PE\0\0", architecture, point d'entrée
├───────────────────┤
│  Section Headers      │  ← table décrivant chaque section
├───────────────────┤
│  .text                 │  ← code
│  .data                 │  ← données initialisées
│  .rdata                │  ← données en lecture seule + Import Table souvent ici
│  .bss                  │  ← données non initialisées
│  .rsrc                 │  ← ressources (icônes, strings, dialogues...)
│  .reloc                │  ← informations de relocation (pour ASLR)
└───────────────────┘
```

Outils : **PEview**, **CFF Explorer** (inspection visuelle), ou `objdump -x` sous Linux sur un fichier PE.

---

## 5. PLT/GOT (ELF) vs Import Table/IAT (PE) — le mécanisme d'appel externe

### Côté Linux (ELF)

Quand ton programme appelle `printf()` (une fonction de la libc, externe), l'appel ne va pas directement à l'adresse de `printf` — il passe par deux structures :

- **PLT** (_Procedure Linkage Table_) : un petit bout de code "tremplin" dans `.text`, appelé à la place de la vraie fonction
- **GOT** (_Global Offset Table_) : une table de pointeurs, remplie **au premier appel** (lazy binding) avec la vraie adresse de `printf` en mémoire

```
CALL printf@plt
      │
      ▼
PLT stub: JMP [GOT entry for printf]
      │
      ▼
GOT entry: adresse réelle de printf (résolue au runtime par le linker dynamique)
```

**Pourquoi c'est important en sécurité** : la technique d'exploitation **GOT overwrite** consiste à écraser une entrée de la GOT pour rediriger un appel de fonction légitime vers du code contrôlé par l'attaquant — une technique classique quand la protection NX empêche l'exécution directe de shellcode.

### Côté Windows (PE)

Le mécanisme équivalent est l'**IAT** (_Import Address Table_) — une table remplie par le loader Windows au chargement avec les vraies adresses des fonctions importées (`CreateFileA`, `VirtualAlloc`...). La technique équivalente d'exploitation/instrumentation s'appelle l'**IAT hooking**.

---

## 6. Sections = permissions mémoire (lien direct avec l'exploitation)

Chaque section a des droits **RWX** (Read/Write/eXecute) définis dans le header :

|Section|Permissions typiques|Pourquoi|
|---|---|---|
|`.text`|R-X (lecture + exécution, **pas** écriture)|Le code ne doit pas pouvoir être modifié à l'exécution|
|`.data` / `.bss`|RW- (lecture + écriture, **pas** exécution)|Ce sont des données, pas du code|
|`.rodata` / `.rdata`|R-- (lecture seule)|Constantes immuables|

Cette séparation stricte (particulièrement **"pas exécutable" sur les données**) s'appelle la protection **NX** (_No-eXecute_, ou **DEP** _Data Execution Prevention_ sous Windows). Elle empêche l'attaque naïve consistant à injecter du shellcode dans un buffer sur la pile puis à sauter dessus — puisque la pile n'est pas exécutable. C'est ce qui a poussé au développement de techniques plus avancées comme le **ROP** (Return Oriented Programming), abordé dans le Cours 10.

---

## 7. Autres protections binaires à connaître (aperçu)

|Protection|Rôle|Où la voir|
|---|---|---|
|**Stack Canary**|Valeur aléatoire placée juste avant l'adresse de retour ; vérifiée avant le `RET` — si modifiée (par un overflow), le programme s'arrête|Visible dans le prologue : chargement depuis `fs:0x28` (Linux) puis vérification en fin de fonction|
|**NX/DEP**|Pile et tas non exécutables|Vue dans les permissions de section (voir ci-dessus)|
|**ASLR** (_Address Space Layout Randomization_)|Randomise les adresses de la pile, du heap, des bibliothèques à chaque exécution|Rend les adresses fixes inutilisables d'un run à l'autre|
|**PIE** (_Position Independent Executable_)|Le binaire lui-même (`.text` inclus) est chargé à une adresse aléatoire, pas seulement les libs|Complète l'ASLR pour le code du binaire lui-même|
|**RELRO** (_Relocation Read-Only_)|Rend la GOT en lecture seule après résolution (Full RELRO), empêchant le GOT overwrite|Spécifique ELF|

Ces protections ne s'excluent pas — un binaire moderne cumule généralement Canary + NX + ASLR/PIE + Full RELRO, ce qui rend l'exploitation bien plus complexe qu'un simple overflow d'adresse de retour (voir méthodologie complète au Cours 10).

Outil pratique pour vérifier ces protections en un coup d'œil : `checksec ./binaire` (à installer via `pip install pwntools` qui inclut `checksec`, ou le paquet dédié).

---

## 8. Ce qu'il faut retenir

1. ELF et PE ne changent **pas** les instructions assembleur elles-mêmes (mêmes MOV/CMP/JMP en x86), mais organisent et chargent le programme différemment.
2. `.text` = code, `.data`/`.bss` = variables, `.rodata`/`.rdata` = constantes — savoir où chercher quoi.
3. PLT/GOT (Linux) et IAT (Windows) gèrent les appels de fonctions externes — cibles classiques d'exploitation.
4. Les protections modernes (Canary, NX, ASLR/PIE, RELRO) se cumulent et déterminent la difficulté réelle d'un exploit — `checksec` est le premier réflexe avant toute tentative.

**Prochain cours** : la méthodologie complète de reverse engineering et l'exploitation (buffer overflow, stack overflow, et le cas du web) — avec le cadre légal/éthique associé.