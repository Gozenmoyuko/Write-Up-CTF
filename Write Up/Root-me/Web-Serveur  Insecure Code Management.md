
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

![](../../Pasted%20image%2020260407222630.png)

Maintenant vous allez tentez de vous déplacer dans chacun des dossiers. Ici je vous montre un qui est intéressant avec /logs/refs/heads/master (Un fichier qui se trouve aussi dans refs mais ce fichier permet de stocker les événements. Ici on peut voir qu'il y a 5 événement qui ont été enregistrer allons voir )

![](../../Pasted%20image%2020260407222759.png)

Okay on peut voir que l'administrateur à initialisé le commit (github) , à sécurisé le mot de passe via MD5, à changé le mot de passe mais aussi à renommé le nom de l'application. Un message du commit à dis comme quoi ils veulent pour le mot de passe du sha256. Peut-être qu'on trouvera un mot de passe sous sha256 ou soit sous md5 s'il n'y as pas encore eu le changement.

Sortons d'ici et continuons de chercher.