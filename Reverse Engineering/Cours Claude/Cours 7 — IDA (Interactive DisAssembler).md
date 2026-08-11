## 1. Qu'est-ce qu'IDA

IDA est l'outil historique de référence pour l'**analyse statique** (lire/comprendre un binaire **sans** l'exécuter). Développé par Hex-Rays, c'est l'outil le plus utilisé dans l'industrie professionnelle (malware analysis, audit de firmware, recherche de vulnérabilités).

| Version      | Prix                                           | Limitations                                                                                           |
| ------------ | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **IDA Free** | Gratuit                                        | Pas de décompilateur, architectures limitées (x86/x64 seulement), pas de scripting avancé             |
| **IDA Pro**  | Payant (licence chère, plusieurs milliers d'€) | Tout débloqué : décompilateur Hex-Rays, toutes architectures (ARM, MIPS...), scripting Python complet |

Pour débuter, **IDA Free** suffit largement — l'essentiel (désassemblage, xrefs, renommage) est disponible.

---

## 2. L'interface principale

### La vue Disassembly (par défaut)

Affiche le code assembleur linéairement, instruction par instruction, avec :

- Les adresses à gauche
- Les instructions au centre
- Des commentaires auto-générés à droite (ex : quel registre contient quoi)

### La vue Graph (la plus utilisée en pratique)

Transforme le code en **blocs reliés par des flèches**, représentant visuellement les branchements (if/else, boucles). Bascule avec la touche `Space`.

```
┌──────────────┐
│  CMP EAX, 5   │
│  JG  loc_A     │
└──────┬───────┘
   │no        │yes
   ▼            ▼
┌───────┐  ┌───────┐
│ bloc B │  │ bloc A │
└───────┘  └───────┘
```

**Pourquoi c'est puissant** : tu vois immédiatement la structure logique (if/else, boucles) sans avoir à suivre mentalement chaque JMP dans la vue linéaire.

---

## 3. Fonctionnalités clés à maîtriser

|Fonctionnalité|Raccourci/Usage|Rôle|
|---|---|---|
|**Renommer** (registre, variable, fonction)|`N`|Remplace `sub_401136` par un nom clair comme `check_password` — essentiel pour garder une trace de ta compréhension|
|**Commenter**|`:` (colon)|Ajoute une note à une ligne précise|
|**Xrefs (cross-references)**|`X` sur un élément|Montre **qui appelle** cette fonction / **qui utilise** cette variable — indispensable pour remonter les dépendances|
|**Strings window**|`Shift+F12`|Liste toutes les chaînes de caractères du binaire — souvent le point de départ (chercher "password", "error", des messages d'erreur...)|
|**Functions window**|`Shift+F3`|Liste toutes les fonctions détectées|
|**Hex View**|`Shift+F2` (ou synchronisé automatiquement)|Vue hexadécimale brute, synchronisée avec la vue désassemblage|
|**Imports/Exports window**|menu View → Open subviews|Liste des fonctions importées/exportées (voir Cours 9 sur ELF/PE)|
|**Hex-Rays Decompiler** (IDA Pro seulement)|`F5`|Transforme l'assembleur en **pseudo-code façon C**, énormément plus rapide à lire|

---

## 4. Méthodologie de base pour analyser un binaire inconnu avec IDA

1. **Ouvrir le binaire** — IDA analyse automatiquement (auto-analysis) et détecte les fonctions
2. **Regarder les Strings** (`Shift+F12`) — souvent le point d'entrée le plus rapide pour comprendre le but du programme
3. **Faire un clic-droit → Jump to xref** depuis une string intéressante pour voir où elle est utilisée
4. **Remonter la fonction** en Graph view, renommer les variables/fonctions au fur et à mesure de ta compréhension
5. **Suivre les xrefs** des fonctions importantes pour comprendre l'appel global du programme

---

## 5. Le scripting IDAPython

IDA embarque un interpréteur Python permettant d'automatiser des tâches (utile vu ton profil Python) :

```python
import idautils
import idc

# Lister toutes les fonctions du binaire
for func_ea in idautils.Functions():
    name = idc.get_func_name(func_ea)
    print(f"{hex(func_ea)}: {name}")
```

Utile pour de l'analyse à grande échelle (ex : identifier automatiquement des patterns de fonctions suspectes dans un malware).

---

## 6. IDA vs les alternatives — quand utiliser quoi

|Situation|Outil recommandé|
|---|---|
|Analyse statique rapide, tu veux le meilleur confort visuel|IDA (Free suffit pour x86/x64)|
|Besoin d'un décompilateur gratuit|**Ghidra** (voir Cours 8)|
|Analyse en ligne de commande, scripting rapide|**radare2** (voir Cours 8)|
|Analyse dynamique, exécution pas à pas|**GDB** (Cours 6)|

En pratique, beaucoup de reversers combinent : Ghidra ou IDA pour l'analyse statique + GDB pour vérifier dynamiquement les hypothèses.

**Prochain cours** : Ghidra et radare2, les alternatives gratuites et open-source.