
Dans ce challenge notre objectif et de lire le fichier index.php. 

![](../img/Pasted%20image%2020260421202839.png)

Pour cela nous allons devoir utiliser l'extension Zip qui est le seul fichier accepter dans le POST. 

Voyons voir comment est le site  : 

![](../img/Pasted%20image%2020260421202412.png)

Le site ressemble à cela, c'est un site plutôt simple, essayons de voir comment ce comporte le site lorsqu'on upload un zip pour cela je vous invite à aller sur votre terminal : 

![](../img/Pasted%20image%2020260421202640.png)

Ici je créé tout d'abord un espace pour mon exploit via la commande mkdir, par la suite je rentre dans le dossier et je vais créé un fichier exploit.php qui contient : 

![](../img/Pasted%20image%2020260421202803.png)

Ici je vais tenter de lire le fichier index.php, pour l'instant je ne sais pas si je suis dans un autre dossier upload et dans combien de répertoire enfant je suis donc je laisse ceci. 

maintenant je vais faire un zip de ce fichier et de le mettre sur le site : 

![](../img/Pasted%20image%2020260421202959.png)
Parfait il à été créé maintenant je vais le mettre sur le site.

![](../img/Pasted%20image%2020260421203045.png)

J'ai mis mon fichier et j'ai cliquer sur Submit, maintenant on me dis que je peux avoir accès à mon fichier zip décompresser en cliquant sur "here" c'est ce que je vais faire : 

![](../img/Pasted%20image%2020260421203138.png)

Okay nous pouvons voir que mon fichier zip à été renommé mais sinon on peut voir que mon fichier .php est bel est bien ici, essayons de cliquez dessus ! 

![](../img/Pasted%20image%2020260421203213.png)

Bon bah il va falloir trouver un autre moyen, nous n'avons pas accès au fichier... 

Après de multiple recherche j'ai trouver ce site qui nous explique quelques chose d'intéressant qui pourrait nous servir : https://www.freecodecamp.org/news/linux-ln-how-to-create-a-symbolic-link-in-linux-example-bash-command/

Elle nous explique qu'est-ce qu'un symlinks : 

Un lien symbolique(symlinks) est un type de fichier qui pointe vers d'autres fichiers ou répertoires (dossiers) sous Linux.

On nous dis qu'on peut créé des liens symbolique (symlinks) via la commande "ln" 