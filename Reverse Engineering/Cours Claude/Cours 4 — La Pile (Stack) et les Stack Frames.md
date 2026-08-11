
## 1. Qu'est-ce qu'une pile ?

Une pile (stack) est une structure de données **LIFO** : **L**ast **I**n, **F**irst **O**ut — le dernier élément ajouté est le premier retiré. Imagine une pile d'assiettes : tu poses (empiles) par-dessus, tu retires (dépiles) par le dessus aussi.

En Python, c'est l'équivalent de :

```python
pile = []
pile.append(10)   # équivalent de PUSH
pile.append(20)
x = pile.pop()     # équivalent de POP, x = 20
```

## 2. La pile en assembleur x86 — particularité importante

**La pile grandit vers les adresses basses** (à l'inverse de l'intuition "empiler = monter"). Concrètement :

```
Adresses hautes
┌───────────────┐
│  ancien contenu │
├───────────────┤ ← ESP avant le PUSH
│  ancien contenu │
├───────────────┤ ← ESP après PUSH (ESP a DIMINUÉ)
│   nouvelle valeur │
└───────────────┘
Adresses basses
```

- **PUSH** : décrémente ESP (souvent de 4 en 32 bits, 8 en 64 bits), puis écrit la valeur à cette nouvelle adresse.
- **POP** : lit la valeur à l'adresse pointée par ESP, puis incrémente ESP.

**ESP pointe toujours sur le sommet actuel de la pile** — c'est sa seule et unique fonction.

---

## 3. La Stack Frame — l'espace de travail d'une fonction

Chaque fois qu'une fonction est appelée, elle se réserve un espace sur la pile pour ses variables locales, appelé **stack frame** (ou "activation record"). Voici le schéma complet d'une frame typique en x86 32 bits :

```
Adresses hautes
┌─────────────────────────┐
│   Arguments de la fonction  │  ← passés par la fonction appelante (convention cdecl)
├─────────────────────────┤
│   Adresse de retour         │  ← où revenir après RET (empilée automatiquement par CALL)
├─────────────────────────┤ ← EBP pointe ici
│   Ancien EBP (sauvegardé)   │  ← EBP de la fonction appelante
├─────────────────────────┤
│   Variables locales         │  ← buffers, int locaux, etc.
│         ...                  │
└─────────────────────────┘ ← ESP pointe ici (sommet actuel)
Adresses basses
```

### Pourquoi EBP ET ESP si les deux "pointent" vers la pile ?

- **ESP bouge tout le temps** (à chaque PUSH/POP pendant l'exécution de la fonction) — peu pratique comme référence fixe.
- **EBP reste fixe pendant toute la durée de la fonction** — il sert de point de repère stable pour accéder aux variables locales (`[EBP-4]`, `[EBP-8]`...) et aux arguments (`[EBP+8]`, `[EBP+12]`...) sans se soucier des mouvements d'ESP.

**Mnémo** : EBP = la "photo" du sommet de pile prise au début de la fonction. Même si ESP bouge après, EBP garde ce point de référence.

---

## 4. Prologue et épilogue de fonction

Quand un compilateur génère une fonction, il ajoute systématiquement ce code au début (**prologue**) et à la fin (**épilogue**) :

### Prologue (mise en place de la frame)

```asm
PUSH EBP           ; sauvegarde l'EBP de l'appelant
MOV EBP, ESP        ; EBP = nouveau sommet = base de MA frame
SUB ESP, 0x20        ; réserve 0x20 octets pour mes variables locales
```

### Épilogue (nettoyage avant de partir)

```asm
MOV ESP, EBP        ; remet ESP là où était EBP (libère les variables locales)
POP EBP              ; restaure l'EBP de l'appelant
RET                   ; dépile l'adresse de retour et y saute
```

**Astuce** : `LEAVE` fait exactement `MOV ESP,EBP` + `POP EBP` en une seule instruction — tu le verras souvent juste avant un `RET`.

### Ce que fait vraiment `CALL` / `RET`

- `CALL fonction` = équivaut à `PUSH adresse_instruction_suivante` puis `JMP fonction`
- `RET` = équivaut à `POP EIP` (dépile l'adresse et y saute)

C'est ce mécanisme précis — l'adresse de retour empilée juste avant les variables locales — **qui rend le buffer overflow possible** : si une variable locale (un buffer) déborde, elle écrase d'abord l'EBP sauvegardé, puis l'adresse de retour juste au-dessus.

---

## 5. Exemple concret complet

```c
void vulnerable(char *input) {
    char buffer[16];
    strcpy(buffer, input);   // pas de vérification de taille !
}
```

Stack frame au moment de l'appel à `strcpy` :

```
┌─────────────────────┐
│  Adresse de retour      │  ← si on écrit 16+4(EBP)+4 = 24+ octets, ON L'ATTEINT
├─────────────────────┤
│  Ancien EBP              │  ← écrasé après 16 octets
├─────────────────────┤ ← EBP
│  buffer[16]              │  ← si input fait plus de 16 octets, débordement !
└─────────────────────┘ ← ESP
```

Si `input` fait exactement 16 (buffer) + 4 (EBP) + 4 octets contrôlés, ces 4 derniers octets **remplacent l'adresse de retour**. Au `RET`, le CPU saute là où l'attaquant l'a décidé, au lieu de revenir au code appelant normal. C'est le principe fondamental du **stack buffer overflow**, détaillé en profondeur dans le Cours 10.

---

## 6. Stack vs Heap — la distinction essentielle

||Stack (pile)|Heap (tas)|
|---|---|---|
|Allocation|Automatique (variables locales)|Manuelle (`malloc`/`free` en C, objets Python)|
|Vitesse|Très rapide|Plus lente (gestion par l'allocateur)|
|Taille|Limitée (souvent quelques Mo)|Beaucoup plus grande|
|Durée de vie|Libérée automatiquement à la fin de la fonction|Reste allouée jusqu'à `free()` explicite|
|Croissance|Vers les adresses basses|Vers les adresses hautes|
|Bugs typiques|Stack overflow, écrasement d'adresse de retour|Heap overflow, use-after-free, double-free|

**Mnémo pour Python/C** : en Python, tu n'as jamais à gérer ça — le ramasse-miettes (garbage collector) s'occupe du heap pour toi. En C, `malloc()` alloue sur le heap, et **tu dois appeler `free()` toi-même** — oublier de le faire crée une fuite mémoire, le faire deux fois crée un `double-free` exploitable.

---

## 7. Ce qu'il faut retenir avant la suite

1. La pile grandit vers le bas ; ESP = sommet mobile, EBP = base fixe de la frame courante.
2. Chaque fonction a un prologue (`PUSH EBP` / `MOV EBP,ESP` / `SUB ESP,N`) et un épilogue (`LEAVE` / `RET`).
3. L'adresse de retour est empilée juste au-dessus de l'EBP sauvegardé, lui-même juste au-dessus des variables locales — c'est cette proximité qui rend le débordement de buffer dangereux.
4. Stack = automatique et rapide, Heap = manuel et plus flexible — deux familles de bugs mémoire différentes.

**Prochain cours** : les conventions d'appel — comment les arguments et la valeur de retour circulent entre fonctions.