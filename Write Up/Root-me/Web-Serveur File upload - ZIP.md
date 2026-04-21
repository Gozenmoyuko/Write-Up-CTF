
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

l'option -s permet de faire un lien relatif si on ne met pas l'option -s cela fait une redirection avec le chemin depuis la racine, or nous voulons partir du relatif pour remonté vers index.php on va faire du path traversal mais d'une autre façon on va dire. 

![](../img/Pasted%20image%2020260421203920.png)

Tout d'abord il va falloir supprimé le fichier zip de base, car le symlinks doit être créé avant le zip vous verrez pourquoi. 

Créons maintenant un symlinks(lien symbolique) comme il nous à été expliquer, voici comment taper la commande : 

ln -s (notre fichier.php) (notre fichier symlinks).(l'extension accepter)

Mais comment connaître l'extension accepter ? J'ai tout simplement fait refait le zip avec un fichier test.txt dans lequel j'ai mis un texte test et j'ai regarder si le serveur me renvoie toujours error 403. 

![](../img/Pasted%20image%2020260421204424.png)

J'ai remis sur le site et appuyer sur sumbit :

![](../img/Pasted%20image%2020260421204452.png)

Je vais tenter de cliquer sur test.txt 

![](../img/Pasted%20image%2020260421204515.png)

Okay parfait on peut lire les fichiers .txt, on va donc mettre la commande suivante : 

ln -s exploit.php exploit.txt


![](../img/Pasted%20image%2020260421204609.png)

On peut voir grâce à ls que notre fichier exploit.txt redirige bien vers exploit.php sauf qu'on ne veut pas qu'il redirige vers exploit.php mais vers notre index.php qui se situe à la racine du site. 

On est actuellement dans : 
http://challenge01.root-me.org/web-serveur/ch51/tmp/upload/69e7c537419dc5.37161142/
![](../img/Pasted%20image%2020260421204727.png)
Il faut remonté de 3 niveau on va donc mettre à la place de notre exploit.php maintenant que vous êtes familier vous devriez savoir qu'on va rediriger vers ../../../index.php ce qui va faire que lorsqu'on va cliquer sur le fichier exploit.txt celui-ci va nous renvoyer le code source index.php qui se trouve à la racine : 

![](../img/Pasted%20image%2020260421210523.png)

On peut remarqué ici que exploit.txt nous redirigera bien vers ../../../index.php ce qui nous renverra vers la racine.

Vous pouvez voir que j'ai eu une erreur car oui si on se base sur le man de zip qui nous ai donnée dans les ressources pour l'exo on trie avec "symlinks" et on voit qu'il faut faire l'option -y car zip prend en charge de base que des fichiers concret et qu'il faut qu'on lui disent si notre fichier qu'on met dans le zip est un symlinks (lien symbolique) si c'est le cas alors il faut mettre l'option -y. 

Bon essayons d'upload  exploit.zip sur le site ! 

![](../img/Pasted%20image%2020260421210815.png)

Moment de vériter je vais cliquer sur exploit.txt : 

![](../img/Pasted%20image%2020260421210910.png)

Bingo ! 
![](../img/Pasted%20image%2020260421210949.png)

Source utile aussi qu'on aurait pu utiliser :

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Zip%20Slip
