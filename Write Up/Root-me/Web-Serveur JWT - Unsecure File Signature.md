
Dans ce challenge il nous faudra trouver un flag sur le site. 

Voici l'énoncer : 

![](../img/Pasted%20image%2020260417235555.png)

Avant de commencer, je vous invite à faire les anciens challenges liée JWT ou lire au moins mes Write-up à ce sujet qui explique comment marche un token JWT. 

On peut voir dans l'énoncer qu'il y a écrit de manière subtil (kid) dans un token JWT cela signifie Key id, kid est un champs de le header qui permet de d'indiquer qu'elle clé doit être utiliser pour vérifier la signature du token. Globalement le serveur va charger une clé à partir d'un fichier (dans le cas de kid) et va permettre au serveur de vérifier la signature. 

Si celui-ci est bonne, alors la connexion est établie et laisse faire l'action. 

Sinon il y a une erreur. 

Attention si celle-ci est mal implémenté. On peut faire facilement des injections de commandes tout comme du path traversal, - Injection de chemin de fichier..

Je vous invite à regarder la vidéo de root-me qui explique plutôt bien le sujet (mieux que moi je pense mdr) https://www.youtube.com/watch?v=d7wmUz57Nlg

Bon maintenant que l'ont sais cela regardons à quoi ressemble le site. 


![](../img/Pasted%20image%2020260418000621.png)


À part le fait que le site est sacrément moche, on peut essayer de voir quelques endpoint comme par exemple le dashboard de base de notre utilisateur root-me (en haut à droite). 
Cliquez sur le petit bonhomme. 


![](../img/Pasted%20image%2020260418000848.png)


![](../img/Pasted%20image%2020260418000840.png)

Tiens intéressant. Un url avec admin :) 

Essayons de l'intercepter via burpsuite. Vous devez être habitué maintenant avec tout les challenges que nous avons fait ensemble haha. 

![](../img/Pasted%20image%2020260418000949.png)

Bon on peut voir que mon token JWT n'est pas admin et que le site nous le fait clairement savoir dans ça response. 

Essayons de voir les trois champs de notre JWT sur le site : https://www.jwt.io. 

![](../img/Pasted%20image%2020260418001142.png)

À gauche j'ai simplement mis le token qui m'était donné et on peut bel est bien voir que je ne suis pas admin. 

Déjà nous allons y remédier. En changeant dans le payload la valeur du "user" en admin. 

Pour cela rendez vous dans la section JWT Encoder. 


![](../img/Pasted%20image%2020260418001458.png)

Dès que cela est fait, nous allons tenter de modifier la valeur de KID ici vous pouvais voir que le header à déjà une valeur (c'est juste la version de kid qui est un fichier directement). Bon essayons de mettre le token admin maintenant avant de passer à la modification du header kid.

![](../img/Pasted%20image%2020260418001534.png)

Bon nous avons pas le bon signature mais c'est pas grave nous allons tenter de le bypass via du Path traversal. Pour cela on va changer notre kid en quelques chose que l'ont connaît. Plusieurs façon sont possible, tel que prendre le retour d'une page WEB déjà sur le site pour mettre la valeur de retour. Mais ici je vais utiliser /dev/null qui est un fichier qui ne contient rien sur linux. En gros il ne contient aucune donnée, tout ce qui rentre est directement supprimé et on va utiliser cela. Car lors que notre kid va chercher une valeur de clé null et va tenter de vérifier grâce à cela notre signature et bien on as juste à mettre une signature null et cela sera bon. 