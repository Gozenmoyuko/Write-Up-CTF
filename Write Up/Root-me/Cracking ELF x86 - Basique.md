
Bonjour aujourd'hui nous allons faire le deuxième challenge de root-me sur la partie cracking. Pour l'instant voici l'énoncer : 

![](../img/Pasted%20image%2020260812012634.png)

Le but est de trouver le mot de passe du challenge. Pour cela on va lancer et voir comment ce comporte le code. Ici j'utiliserais GDB ajouté avec une extension nommée peda qui est simple d'installation voici le github du projet peda : 

https://github.com/longld/peda

Je lance tout d'abord le programme normalement : 

![](../img/Pasted%20image%2020260812012848.png)

Pour trouver le processus : 

![](../img/Pasted%20image%2020260812012525.png)


![](../img/Pasted%20image%2020260812001446.png)

Ici j'exécute entant que root pour l'option -E permet d'utiliser l'environnement actuel et pas celui du root. Je met le code source du code donc le binaire et le process id (PID) que va se rattacher GDB pour faire son mode de debugage.

![](../img/Pasted%20image%2020260812001427.png)

Ici nous pouvons voir plusieurs registre mais aussi le code source associés avec les instructions assembleurs comme par exemple cmp qui permet de comparer deux valeurs de registre donc par exemple un input d'utilisateur avec une valeur que l'ont à défini auparavant. Nous avons aussi par exemple tous ce qui va être dans la catégorie stack qui sont les adresses en mémoire vive. 

Tout en haut dans les registre on peu tout d'abord voir plusieurs registre ce qui m'interesse ici c'est le registre EIP qui est le registre de la prochaine instruction. Extended Instruction Pointer (EIP)

On va tout d'abord donc mettre en pause le processus main pour après le désassembler pour comprendre un peu plus ce que fais le code source.

![](../img/Pasted%20image%2020260812001340.png)

On peut voir que celui-ci effectue plusieurs fonctions par exemple le puts qui doit être pour afficher un texte. Moi je vais m'interessais ici sur la fonction strcmp qui si le développeur est censer veut dire que la fonction compare deux chaînes de caractère.

Je vais donc rentrer dans la fonction strcmp en faisans disas strcmp.

![](../img/Pasted%20image%2020260812001248.png)
Ici nous pouvons voir qu'il y a plusieurs test qui sont fais entre plusieurs variables mais aussi des réécriture dans d'autre variable etc. Ce qui va nous intéresser ici c'est les sauts vers d'autre instructions ou adresse mémoire. On rappel que chaque adresse mémoire contiens une valeur, si celle-ci n'as pas était défini dans le code (en c avec maloc ou autre) alors celui-ci prends la valeurs qui était stocker avant dans cette adresse mémoire. Ici on voit qu'il y a un JE (Jump Equal) donc si les deux variables (dans le deuxième JE et dans le deuxième al, cl) bref il vont comparer caractère par caractère et regarde si c'est égale. Si cela est le cas alors il saute à la prochaine instruction. Pour les petits curieux, le test al, al permet de voir s'il y a le caractère de fin de chaîne qui est \0 ou que si le registre al = 0.

Vous pouvez voir que j'ai notamment taper : 
set {char}[Adresse en hexa]=[opcode instruction assembleur]

Pour ceux qui sont pas habitué à qu'est-ce qu'un opcode, enfaite un opcode c'est une instruction assembleur qui est encoder en hexadécimal, vous pouvez facilement les trouvés lorsque vous aurez besoin ou même faites vous une fiche globale (il y en as beaucoup je privilégie la recherche personnellement.) Maintenant que vous savez enfaite ce que j'ai fais c'est que j'ai remplacer ducoup 1 octet à l'adresse mémoire en lui affectant le nouveau opcode. Ce qui veut dire que j'ai inversé la logique du code. Maintenant si je n'ai pas le bon username alors je passe à la suite. 
Résultat : 
```asm
set {char}0x805030a=0x75
```

*NB : 0x74 = JE, 0x75 = JNE*

Puis je fais un point d'arrêt sur l'adresse mémoire pour voir ce qui se passe exactement. C'est le fameux 
```
b *[adresse_en_hexa]
```

Cela va faire un arrêt sur le processus lorsqu'il va sauté à cette instruction.

![](../img/Pasted%20image%2020260812012343.png)

J'ai donc marquer test et comme nous pouvons le voir en dessous il y a d'autre informations utilisateur (que j'ai enlever sinon vous auriez déjà la réponse.)

![](../img/Pasted%20image%2020260812001208.png)

J'arrête l'attachement au processus et je test avec les données utilisateurs trouver.


![](../img/Pasted%20image%2020260812001647.png)