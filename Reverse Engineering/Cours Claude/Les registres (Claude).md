
Pareil que sur le cours de base sur le reverse je ferai un cours de moi même mais voici une fiche pour vous aider lorsque vous allez essayer de faire du reverse solo et de pas trop vous perdre.

# Fiche mémo — Registres Assembleur x86 / x86-64

## 1. Le principe des préfixes (comment se forme un nom de registre)

Un nom de registre = **[Préfixe de taille] + [Racine fonctionnelle] + [Suffixe éventuel]**

|Préfixe/Suffixe|Signification|Époque|Taille|
|---|---|---|---|
|_(rien)_|registre "historique" 16 bits|8086 (1978)|16 bits|
|**E**|**E**xtended (étendu)|386 (1985)|32 bits|
|**R**|juste "**R**egister" (extension x86-64)|x86-64 (2003)|64 bits|
|**L** (suffixe)|**L**ow byte (octet bas)|—|8 bits|
|**H** (suffixe)|**H**igh byte (octet haut)|—|8 bits|

**Mnémo** : à chaque génération de CPU, Intel a "étendu" (**E**xtended) puis re-étendu (**R**) les mêmes registres de base sans changer leur rôle. C'est pour ça que `AX → EAX → RAX` désignent la même fonction, juste avec plus de bits.

Exemple complet sur l'accumulateur :

```
AL/AH (8 bits) → AX (16 bits) → EAX (32 bits) → RAX (64 bits)
```

---

## 2. Les 8 registres généraux (GPR) — la racine à 2 lettres

|Registre 32b|Racine|Décomposition|Rôle historique / usage actuel|
|---|---|---|---|
|**EAX**|**A**ccumulator|E=Extended, A=Accumulator|Résultats arithmétiques, **valeur de retour** d'une fonction|
|**EBX**|**B**ase|E=Extended, B=Base|Registre "libre", pointeur de **base** pour adresser la mémoire|
|**ECX**|**C**ounter|E=Extended, C=Counter|**Compteur** de boucles (l'instruction `LOOP` décrémente ECX automatiquement)|
|**EDX**|**D**ata|E=Extended, D=Data|**Données** complémentaires (multiplication/division étendue, I/O)|
|**ESI**|**S**ource **I**ndex|E=Extended, S=Source, I=Index|Pointeur **source** pour copier des chaînes/tableaux|
|**EDI**|**D**estination **I**ndex|E=Extended, D=Destination, I=Index|Pointeur **destination** pour copier des chaînes/tableaux|
|**EBP**|**B**ase **P**ointer|E=Extended, B=Base, P=Pointer|Pointe la **base fixe** de la pile (frame de la fonction)|
|**ESP**|**S**tack **P**ointer|E=Extended, S=Stack, P=Pointer|Pointe le **sommet mobile** de la pile|

### Moyen mnémotechnique global

Pense à la phrase : **"A**ccumule, **B**ase, **C**ompte, **D**onnées" (A-B-C-D dans l'ordre alphabétique = les 4 registres "de calcul").

Puis pour SI/DI : imagine un copié-collé →

> "Je pars de la **S**ource (ESI) pour aller vers la **D**estination (EDI)." C'est littéralement ce que fait l'instruction `MOVS` (move string).

Pour EBP/ESP, les deux ont un **P** de **Pointer**, la nuance est dans la 1ère lettre :

- **B**P = **B**ase → fixe, posé au début de la fonction (comme la fondation d'une maison)
- **S**P = **S**tack → bouge à chaque `push`/`pop` (comme le sommet d'une pile d'assiettes)

---

## 3. Les registres spéciaux (pas des GPR)

|Registre|Décomposition|Rôle|
|---|---|---|
|**EIP / RIP**|E/R + **I**nstruction **P**ointer|Adresse de la **prochaine instruction** à exécuter. **C'est LUI que tu cherches à écraser en buffer overflow.**|
|**EFLAGS / RFLAGS**|E/R + FLAGS (drapeaux)|Registre de statut, chaque bit = un "flag"|

### Flags à connaître (dans EFLAGS)

|Flag|Signification|Se déclenche quand...|
|---|---|---|
|**ZF**|**Z**ero **F**lag|le résultat de l'opération = 0|
|**CF**|**C**arry **F**lag|une retenue déborde (dépassement non signé)|
|**SF**|**S**ign **F**lag|le résultat est négatif (bit de poids fort = 1)|
|**OF**|**O**verflow **F**lag|dépassement de capacité en arithmétique signée|
|**PF**|**P**arity **F**lag|nombre pair de bits à 1 dans l'octet de poids faible|

**Mnémo** : toutes les 5 lettres finissent par **F** = **F**lag. Le préfixe te dit exactement ce qu'il surveille (Zero, Carry, Sign, Overflow, Parity) — pas besoin d'apprendre par cœur, juste traduire la lettre.

---

## 4. Découpage en octets (pour EAX, EBX, ECX, EDX uniquement)

```
        31                16 15        8 7         0
EAX  =  [        ]  [        AX        ]
                        [   AH   ][   AL   ]
```

|Partie|Nom|Décomposition|
|---|---|---|
|32 bits entiers|EAX|Extended AX|
|16 bits bas|AX|registre historique 16 bits|
|8 bits haut de AX|AH|**A**X **H**igh|
|8 bits bas de AX|AL|**A**X **L**ow|

Même logique pour B, C, D → BX/BH/BL, CX/CH/CL, DX/DH/DL. (ESI, EDI, EBP, ESP n'ont pas de version H, seulement un accès bas 16 bits type SI, DI, BP, SP — pas de découpage en octets séparés H/L pour ceux-là).

---

## 5. Passage en 64 bits (x86-64)

| 32 bits | 64 bits | Nouveaux registres (pas d'équivalent 32b historique)          |
| ------- | ------- | ------------------------------------------------------------- |
| EAX →   | **RAX** | R8, R9, R10, R11, R12, R13, R14, R15                          |
| EBX →   | **RBX** | (numérotés, pas de nom fonctionnel — juste "Register 8 à 15") |
| ECX →   | **RCX** | Leurs sous-parties : R8D (32b), R8W (16b), R8B (8b)           |
| EDX →   | **RDX** |                                                               |
| ESI →   | **RSI** |                                                               |
| EDI →   | **RDI** |                                                               |
| EBP →   | **RBP** |                                                               |
| ESP →   | **RSP** |                                                               |
| EIP →   | **RIP** |                                                               |

**Mnémo** : R8 à R15 sont numérotés parce qu'AMD (créateur du x86-64) a arrêté d'inventer des noms fonctionnels et a juste dit "**R**egistre n°8, 9, 10...".

---

## 6. Application directe pour toi (pentest / exploit dev)

Puisque tu bosses sur du offensive/CTF, voici pourquoi ces registres comptent concrètement :

- **EIP/RIP** : cible n°1 d'un buffer overflow → si tu l'écrases, tu contrôles le flux d'exécution.
- **ESP/EBP (RSP/RBP)** : délimitent la **stack frame** → essentiels pour calculer l'offset jusqu'au retour de fonction (pattern `cyclic` de pwntools/msf-pattern_create).
- **EAX/RAX** : contient souvent le **syscall number** en x86-64 Linux, ou la valeur de retour d'une fonction — utile en shellcoding.
- **ECX/RCX** : apparaît dans les appels de convention (registres d'arguments en x86-64 System V : RDI, RSI, RDX, RCX, R8, R9 dans cet ordre — mnémo : **"Di-Si-D-C-8-9"**, retiens juste l'ordre des 4 premiers avec ESI/EDI/EDX/ECX que tu connais déjà).
