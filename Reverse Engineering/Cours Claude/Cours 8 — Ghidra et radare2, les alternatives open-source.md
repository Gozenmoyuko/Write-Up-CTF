## Partie 1 — Ghidra

### 1.1 Présentation

Ghidra est un outil de rétro-ingénierie développé par la **NSA**, rendu open-source et gratuit en 2019. Il rivalise directement avec IDA Pro, avec un avantage énorme : **le décompilateur est inclus gratuitement** (contrairement à IDA où Hex-Rays est payant).

### 1.2 Concepts de base

|Notion|Rôle|
|---|---|
|**Project**|Conteneur regroupant un ou plusieurs binaires analysés|
|**CodeBrowser**|La fenêtre principale d'analyse (équivalent de l'interface principale d'IDA)|
|**Listing view**|Vue désassemblage linéaire (comme la vue Disassembly d'IDA)|
|**Decompiler view**|Pseudo-code type C, généré automatiquement — **toujours affiché par défaut**, contrairement à IDA où il faut la version Pro|
|**Symbol Tree**|Arborescence des fonctions, labels, classes détectées|
|**Data Type Manager**|Gestion des structures/types de données que tu définis toi-même|

### 1.3 Workflow de base

1. **Créer un projet** → `File > New Project`
2. **Importer le binaire** → `File > Import File`
3. **Lancer l'analyse automatique** → double-clic sur le fichier importé, dire "Yes" à "Analyze this file now?"
4. Une fois l'analyse terminée, tu as **simultanément** : la vue Listing (assembleur) à gauche/centre et la vue **Decompile** à droite — les deux sont synchronisées, cliquer une ligne dans l'une surligne l'équivalent dans l'autre

### 1.4 Fonctionnalités clés

|Fonctionnalité|Raccourci|Rôle|
|---|---|---|
|Renommer une fonction/variable|`L`|Comme dans IDA, essentiel pour documenter ta compréhension|
|Xrefs|`Ctrl+Shift+F` ou clic-droit → References|Voir qui référence un élément|
|Rechercher une string|`Search > For Strings`|Équivalent de la Strings window d'IDA|
|Graph view|icône dédiée dans la toolbar|Vue en blocs comme IDA Graph|
|Retyper une variable|clic-droit sur la variable dans le Decompiler|Change le type affiché (ex : `int` → `char*`) pour affiner la lecture|

### 1.5 Scripting Ghidra (Java ou Python via Jython/Ghidrathon)

Ghidra propose un script manager pour automatiser des tâches, en Java nativement, ou en Python via l'extension **Ghidrathon**.

---

## Partie 2 — radare2 (et Cutter, son interface graphique)

### 2.1 Présentation

radare2 (souvent abrégé "r2") est un framework de reverse engineering **en ligne de commande**, extrêmement puissant mais avec une courbe d'apprentissage plus raide (syntaxe de commandes très condensée). **Cutter** est l'interface graphique construite par-dessus, pour ceux qui préfèrent une UI façon IDA/Ghidra.

### 2.2 Pourquoi l'apprendre malgré la difficulté

- 100% scriptable et automatisable (parfait pour du traitement en masse, des CTF avec des scripts d'auto-résolution)
- Très léger, fonctionne en SSH sur une machine distante sans interface graphique
- Utilisé massivement dans l'écosystème CTF

### 2.3 Commandes de base

```bash
r2 -A ./binaire     # ouvre le binaire avec analyse automatique (-A)
```

Une fois dans r2 :

|Commande|Rôle|
|---|---|
|`aaa`|Lance une analyse automatique complète (si pas fait avec `-A`)|
|`afl`|**A**nalyze **F**unction **L**ist — liste toutes les fonctions détectées|
|`pdf @ main`|**P**rint **D**isassemble **F**unction — désassemble la fonction `main`|
|`pdf @ 0x401136`|Désassemble à partir d'une adresse|
|`s main`|**S**eek — déplace le curseur sur la fonction `main`|
|`s 0x401136`|Se positionne sur une adresse précise|
|`iz`|Liste les strings détectées dans le binaire|
|`ii`|Liste les imports|
|`axt 0x401136`|**A**nalyze **X**refs **T**o — qui référence cette adresse|
|`V`|Passe en mode **visuel** (interface texte interactive, plus confortable)|
|`VV`|Mode visuel **graphique** (équivalent Graph view d'IDA/Ghidra, en ASCII-art)|
|`pdc @ main`|**P**rint **D**ecompiled **C**ode — décompilation (via r2ghidra si installé)|

**Mnémo pour les commandes r2** : la première lettre indique souvent la catégorie —

- `a` = **a**nalyze
- `p` = **p**rint
- `s` = **s**eek (se déplacer)
- `i` = **i**nfo (métadonnées du binaire)
- `V` = **v**isual mode

### 2.4 Cutter (interface graphique de radare2)

Si la ligne de commande pure te freine au début, **Cutter** t'offre toute la puissance de radare2 avec une interface façon Ghidra/IDA (listing, graph view, décompilateur via r2ghidra intégré). Bon compromis pour apprendre radare2 progressivement sans tout faire en CLI dès le premier jour.

---

## 3. Tableau comparatif final

|Critère|IDA Pro|IDA Free|Ghidra|radare2 / Cutter|
|---|---|---|---|---|
|Prix|Payant (cher)|Gratuit|Gratuit|Gratuit|
|Décompilateur|Oui (Hex-Rays)|Non|Oui (gratuit)|Oui (via r2ghidra)|
|Interface|Graphique|Graphique|Graphique|CLI (Cutter = graphique)|
|Architectures supportées|Toutes|x86/x64 seulement|Toutes|Toutes|
|Courbe d'apprentissage|Moyenne|Moyenne|Moyenne|Élevée (CLI) mais très puissant une fois maîtrisé|
|Usage recommandé|Standard pro/industrie|Débuter gratuitement|Alternative complète gratuite|Scripting/automatisation, CTF|

**Recommandation pour débuter concrètement** : commence par **Ghidra** (gratuit, décompilateur inclus, interface accessible), ajoute **GDB+pwndbg** pour l'analyse dynamique, puis explore radare2 progressivement une fois à l'aise — c'est l'outil qui paiera le plus en vitesse une fois maîtrisé, notamment en contexte CTF chronométré comme WorldSkills.

**Prochain cours** : les formats binaires ELF et PE, pour comprendre ce que ces outils analysent réellement.