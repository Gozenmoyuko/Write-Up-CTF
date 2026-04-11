
Dans ce challenge, notre but est de retrouver le mot de passe de l'administrateurs qui est stocker en claire. 


Tout d'abord voyons voir comment est le site. 

![](Write%20Up/img/Pasted%20image%2020260407131729.png)


On peut se dire "Ok bah c'est peut-être une faille du code source et on à juste à regarder le code source de base. "

Regardons de plus près. 

![](Write%20Up/img/Pasted%20image%2020260407131816.png)

C'est un simple POST et si j'essaye avec username = admin et password = admin voici ce qui est retourné. 

![](Write%20Up/img/Pasted%20image%2020260407131848.png)

Rien de très intéressant n'est-ce pas ? 


Pour vous faire gagner du temps, sachez que j'ai cherché à faire du path traversal et autre pour trouver des fichiers sensible. Malheureusement sans réussite. Normal on est sur un challenge WEB serveur. 

Essayons de voir s'il existe une faille expliquez qui porte ce nom sur le github [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master)


[PayloadsAllTheThings/Insecure Source Code Management at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Source%20Code%20Management)

Parfait, Essayons d'avoir accès à .git, en gros la faille c'est que l'administrateur n'as pas enlever l'accès au .git, ce qui veut dire qu'on peut avoir toute la structure du site et les fichiers hébergé. 


![](Write%20Up/img/Pasted%20image%2020260407133233.png)

Ici on peut voir que j'ai accès au .git et donc à tout ce qui est hébergé sur le site web.

Mais voyons voir les différents fichier. Il faut qu'on trouve un mot de passe écrit en clair vous vous rappelez ? 

![](../img/Pasted%20image%2020260407222630.png)

Maintenant vous allez tentez de vous déplacer dans chacun des dossiers. Ici je vous montre un qui est intéressant avec /logs/refs/heads/master (Un fichier qui se trouve aussi dans refs mais ce fichier permet de stocker les événements. Ici on peut voir qu'il y a 5 événement qui ont été enregistrer allons voir )

![](../img/Pasted%20image%2020260407222759.png)

Okay on peut voir que l'administrateur à initialisé le commit (github) , à sécurisé le mot de passe via MD5, à changé le mot de passe mais aussi à renommé le nom de l'application. Un message du commit à dis comme quoi ils veulent pour le mot de passe du sha256. Peut-être qu'on trouvera un mot de passe sous sha256 ou soit sous md5 s'il n'y as pas encore eu le changement.

Sortons d'ici et continuons de chercher.

![](../img/Pasted%20image%2020260407223530.png)

En cherchant beaucoup, dans le fichier config ou le head par exemple j'ai vu que beaucoup renvoyer à ce fichier donc au /.git/refs/heads/master

On peut voir que quelque chose ressemble à un mot de passe hashé. Rappelez vous la blue team demander du sha256 et qu'elle autre hash et faible comparer au sha256 ? et oui le sha1 bon testons de trouver le mot de passe.

Nous allons tester de le faire via hashcat pour cela prenons leur docu. 

https://hashcat.net/wiki/doku.php?id=example_hashes

On peut voir sur la page que SHA1 sans salt ni rien est la méthode 100. Mais en quoi cela nous interesse ? car dans hashcat il faut préciser le type de hash que vous voulez cracker via la méthode -m qu'on utilise beaucoup pour les challenges réseau par exemple.

Bon continuons notre commande hashcat va ressembler à cela. 

hashcat -m 100 -a 0  (le_fichier_où_il_y_a_le_hash) /usr/share/wordlists/rockyou.txt


ici -a permet de dire le vecteur d'attaque que l'ont veut faire voilà un exemple.

  # `| Mode`
 `===+======`
  `0 | Straight`
  `1 | Combination`
  `3 | Brute-force`
  `6 | Hybrid Wordlist + Mask`
  `7 | Hybrid Mask + Wordlist`
  `9 | Association`

Ici je pense que la méthode 0 suffira et wordlist c'est sur les mot de passes qu'il va se passé comme une bibliothèque. 


Bon malheureusement hashcat n'as pas trouvé... 

Pour mieux voir et savoir si je me suis pas tromper de piste je vais installer le site grâce à : 

![](../img/Pasted%20image%2020260407225752.png)

On peut voir que tout à été installer voyons voir ce qu'il y a dans mes fichiers. 


Maintenant qu'on à installer le site nous pouvons simplement utiliser les commandes de git que vous pouvez retrouvez sur ce site : 

https://git-scm.com/docs

Nous allons utiliser git log pour voir les logs. 



![](../img/Pasted%20image%2020260407230256.png)Bon on peut voir que c'est les mêmes log.

Bon il faut qu'on trouve le mot de passe encore ! 

Maintenant nous allons utilisé la commande `git status`, cela va nous permettre de voir les fichiers qui ont été supprimé ou modifier. 

![](../img/Pasted%20image%2020260407231236.png)

Maintenant nous allons récupérer le fichier config.php. Pour cela nous allons utiliser la commande git restore [notre fichier] donc ici ce sera : 

	git restore config.php

Maintenant qu'il est restauré essayons de l'ouvrir : 

![](../img/Pasted%20image%2020260407231832.png)

Récupérons se hash et mettons le sur ce site : 

https://hashes.com/en/decrypt/hash

![](../img/Pasted%20image%2020260407231926.png)

Bingo !! 