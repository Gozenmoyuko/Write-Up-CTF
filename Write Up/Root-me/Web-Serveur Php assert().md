
Dans ce challenge notre but est d'exécuté du code arbitraire pour lire un fichier .passwd

![](../img/Pasted%20image%2020260421150358.png)

Vous pouvez lire la documentation qui est juste en dessous si cela vous intéresse elle est très bien si vous savez lire l'anglais.

Voyons voir maintenant le site sur lequel nous devons attaquer le côté serveur.

Voici à quoi ressemble le site : 

![](../img/Pasted%20image%2020260421151235.png)

Nous pouvons voir qu'il y a plusieurs pages disponibles, essayons de voir l'url s'il détient un paramètre http pour peut-être réussir à faire du Path traversal. Qui sait peut-être qu'il n'y a même pas besoin d'exécuté du code arbitraire nan ? 

![](../img/Pasted%20image%2020260421151337.png)

L'url ressemble à ceci. 

Bon tentons simplement de voir si on peut lire directement le fichier .passwd

![](../img/Pasted%20image%2020260421151423.png)

Bon le fichier n'existe pas mais pourquoi ? Pourtant il nous ai dis qu'il faut le trouver. Mais attend j'ai bien mis : http://challenge01.root-me.org/web-serveur/ch47/?page=.passwd

Pourquoi il me dis qu'il ne trouve pas le fichier includes/.passwd.php

Peut-être que dans le code php il rajoute .php à la fin pour évité qu'on n'ouvre autre chose via le paramètre HTTP que du code .php 

Essayons de faire du path traversal maintenant en faisant .. 

![](../img/Pasted%20image%2020260421151637.png)

Tiens tiens tiens, nous allons t'avoir bientôt je pense! 

Ici on peut voir une erreur du à un assert, on peut même voir la forme du code, c'est pas pro cela il faut eviter les assert en php tout le monde le sais :) 

Car maintenant on voit qu'est-ce qu'exécute le serveur à peut près comme protection. 

Il utilise un assert via strpos qui permet de chercher la première occurrence dans une chaîne de caractère, il filtre donc le str et mes ce qu'on met en str. 

Voici la docu de strpos : 
https://www.php.net/manual/fr/function.strpos.php

par la suite dès qu'il trouve une occurrence tel que ".." ce trouve dans le input (l'entrer) de l'utilisateur il donne une valeur, cette valeur est celle où se trouve l'occurrence du premier caractère qui pose problème. Ici avec : 

http://challenge01.root-me.org/web-serveur/ch47/?page=../.passwd

cela sera donc dans notre URL la position 0 mais en réalité le serveur va chercher dans includes/..passwd.php 

Le .php je rappel ce n'est pas nous qui l'avons mis mais le serveur par la suite. 

Ici l'occurrence du premier caractère qui pose problème est le 9 ème. Vous n'avez cas compter à qu'elle position se trouve le premier . du ".." en partant de 0 et vous le verrais. 

Donc strpos renvoie une valeur 9 ce qui est différent de false donc il trouve une occurrence de .. et renvoie l'erreur "Detected hacking attempt!"

Mais grosse erreur de vérifier cela via strpos. Car maintenant nous savons que le Path traversal ne marche pas, donc qu'il faut que notre code ne contiennent pas de ".." ce qui été censé être une correction d'erreur deviens une erreur fatale, puisque notre input est directement mis dans le assert, on peut simplement de plusieurs façon de fermer le code avant(comme une injection de code JS classique par exemple ou html).

Testons de chercher une page entre des guillemets par exemple 'test'
http://challenge01.root-me.org/web-serveur/ch47/?page='test'

![](../../Pasted%20image%2020260421153927.png)
Tiens tiens tiens, il à eu une erreur lorsqu'il à voulu évaluer le code, ce qu'il veut dire plusieurs choses.

- Le serveur à bel est bien mis notre strings dans le strpos
- Que la structure de ce que nous avons vu auparavant est bonne
- "Faillure evaluating code", Le serveur à tenter d'évaluer le code, on peut passer à la vitesse supérieur.

Maintenant que cela est fait. 

Testons de voir qu'est-ce qui nous permet d'exécuté du code shell directement via php et essayons de le mettre entre '{notre injection}'