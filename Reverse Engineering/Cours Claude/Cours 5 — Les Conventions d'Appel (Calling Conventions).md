## 1. Pourquoi ça existe

Quand une fonction en appelle une autre, il faut un accord commun sur :

- **Où** placer les arguments (registres ? pile ? les deux ?)
- **Qui** nettoie la pile après l'appel (l'appelant ou l'appelé ?)
- **Où** se trouve la valeur de retour

Cet accord s'appelle une **convention d'appel** (calling convention). Elle change selon l'OS et l'architecture — c'est une des premières choses à identifier quand tu reverses un binaire, car ça te dit directement où chercher les arguments d'une fonction.

---

## 2. x86 32 bits — les conventions historiques

### cdecl (C declaration) — la plus courante en C

- Arguments passés **sur la pile**, empilés de **droite à gauche**
- **L'appelant** nettoie la pile après l'appel (`ADD ESP, N` après le `CALL`)
- Valeur de retour dans **EAX**

```c
int add(int a, int b) { return a + b; }
add(3, 5);
```

```asm
PUSH 5          ; b empilé en dernier... mais en premier dans le code !
PUSH 3          ; a empilé en dernier
CALL add
ADD ESP, 8       ; L'APPELANT nettoie (2 arguments x 4 octets)
```

**Mnémo** : "c**dec**l" comme "**dec**laration C standard" — c'est la convention par défaut du langage C. Reconnaissable au `ADD ESP, N` juste après le `CALL` (signe que l'appelant nettoie).

### stdcall — utilisée par l'API Windows (WinAPI)

- Arguments sur la pile, droite à gauche (comme cdecl)
- **L'appelée** nettoie la pile (via `RET N` au lieu d'un simple `RET`)
- Valeur de retour dans EAX

**Mnémo** : "**std**call" = "**standard** de Windows" — tu la reconnaîtras à un `RET 0x8` (ou autre valeur) au lieu d'un `RET` simple.

### fastcall — pour les appels fréquents

- Les 2 premiers arguments dans **ECX** et **EDX**, le reste sur la pile
- L'appelée nettoie la pile

**Mnémo** : "**fast**call" = plus rapide car évite d'empiler les premiers arguments.

---

## 3. x86-64 — les conventions modernes (celles que tu croiseras le plus aujourd'hui)

### System V AMD64 ABI — Linux, macOS, BSD

Les 6 premiers arguments entiers/pointeurs passent par des **registres**, dans cet ordre précis :

|Ordre|Registre|
|---|---|
|1er argument|**RDI**|
|2e argument|**RSI**|
|3e argument|**RDX**|
|4e argument|**RCX**|
|5e argument|**R8**|
|6e argument|**R9**|
|7e argument et suivants|sur la pile|

Valeur de retour dans **RAX**. Les arguments flottants (`float`/`double`) utilisent **XMM0 à XMM7** séparément.

**Mnémo pour retenir l'ordre** : tu connais déjà RDI (Destination) et RSI (Source) du Cours 3 — pense à la phrase _"**Di**s-moi **Si** tu as les **D**onnées, **C**alcule avec **8** et **9**"_ (Di-Si-D-C-8-9). Ou plus simple : c'est l'ordre exact dans lequel tu les listerais si on te demandait "quels sont les registres généraux les moins utilisés pour l'arithmétique classique" — RDI, RSI, RDX, RCX, puis les nouveaux R8/R9.

### Microsoft x64 (Windows 64 bits)

Différent du System V ! Seulement **4 registres** pour les arguments :

|Ordre|Registre|
|---|---|
|1er argument|**RCX**|
|2e argument|**RDX**|
|3e argument|**R8**|
|4e argument|**R9**|
|5e et suivants|sur la pile|

Valeur de retour dans **RAX**. Particularité Windows : même avec seulement 4 arguments passés par registre, l'appelant doit réserver 32 octets de "**shadow space**" sur la pile (pour que l'appelée puisse y sauvegarder ces registres si besoin).

**Piège fréquent** : si tu reverses un binaire Windows en pensant "System V" par réflexe (RDI/RSI...), tu vas te tromper — vérifie toujours l'OS cible avant d'interpréter les arguments.

---

## 4. Tableau récapitulatif

|Convention|OS/Contexte|1er arg|Qui nettoie la pile|
|---|---|---|---|
|cdecl|C, 32 bits, historique|Pile|Appelant|
|stdcall|WinAPI, 32 bits|Pile|Appelée|
|fastcall|32 bits, optimisation|ECX|Appelée|
|System V AMD64|Linux/macOS 64 bits|RDI|(pas de nettoyage pile pour les 6 premiers)|
|Microsoft x64|Windows 64 bits|RCX|(pas de nettoyage pile pour les 4 premiers)|

---

## 5. Pourquoi c'est essentiel en reverse engineering

Quand tu regardes une fonction désassemblée sans code source :

1. **Identifier la convention** te dit immédiatement où chercher les arguments (RDI/RSI en Linux 64 bits, RCX/RDX en Windows 64 bits, ou la pile en 32 bits).
2. **Le nombre de registres/valeurs poussés avant un CALL** te donne le nombre d'arguments de la fonction appelée, même sans son nom.
3. **En analyse de shellcode/exploitation**, savoir où passer tes arguments à `execve()` ou une syscall dépend directement de la convention System V (RDI, RSI, RDX pour les 3 premiers paramètres d'une syscall Linux x86-64).
4. **Ghidra/IDA déduisent automatiquement les prototypes de fonctions** en analysant quels registres sont lus juste après un `CALL` — comprendre la convention te permet de vérifier/corriger ces déductions.

**Prochain cours** : GDB — le débogueur à maîtriser en premier pour observer tout ça en live.