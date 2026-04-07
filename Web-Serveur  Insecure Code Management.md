
Dans ce challenge, notre but est de retrouver le mot de passe de l'administrateurs qui est stocker en claire. 


Tout d'abord voyons voir comment est le site. 

![](Write%20Up/img/Pasted%20image%2020260407131729.png)


On peut se dire "Ok bah c'est peut-être une faille du code source et on à juste à regarder le code source de base. "

Regardons de plus près. 

![](Pasted%20image%2020260407131816.png)

C'est un simple POST et si j'essaye avec username = admin et password = admin voici ce qui est retourné. 

![](Pasted%20image%2020260407131848.png)

Rien de très intéressant n'est-ce pas ? 


Pour vous faire gagner du temps, sachez que j'ai cherché à faire du path traversal et autre pour trouver des fichiers sensible. Malheureusement sans réussite. Normal on est sur un challenge WEB serveur. 

Essayons de voir s'il existe une faille expliquez qui porte 