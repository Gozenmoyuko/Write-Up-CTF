
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

Bon nous avons pas le bon signature mais c'est pas grave nous allons tenter de le bypass via du Path traversal. Pour cela on va changer notre kid en quelques chose que l'ont connaît. Plusieurs façon sont possible, tel que prendre le retour d'une page WEB déjà sur le site pour mettre la valeur de retour. Mais ici je vais utiliser /dev/null qui est un fichier qui ne contient rien sur linux. En gros il ne contient aucune donnée, tout ce qui rentre est directement supprimé et on va utiliser cela. Car lors que notre kid va chercher une valeur de clé dans un fichier qui ne contient rien cela va forcé une clé vide et va tenter de vérifier grâce à cela notre signature et bien on as juste à mettre une signature null et cela sera bon. 

D'abord occupons nous de mettre une signature null. 

![](../img/Pasted%20image%2020260418002316.png)

Comme vous pouvez le voir ici, un secret JWT ne peux pas être vide, on va donc le bypass via l'option "base64url encoded"

![](../img/Pasted%20image%2020260418002358.png)
Ne vous en faite pas de l'erreur. Mais comment on sais que c'est AA par exemple et non AA== ? 

Tout simplement regardons ce que donne AA== en base 64 : 

![](../img/Pasted%20image%2020260418002518.png)

Okay on voit que cela nous donne null mais pourquoi ? Regardons la table de Base64 de notre bon vieux Wikipédia https://en.wikipedia.org/wiki/Base64 :

![](../img/Pasted%20image%2020260418002719.png)


Okay on peut voir que la valeur de A en base 64 est globalement (0 x 64) en binaire. Car la valeur de la table est 0 par exemple pour faire B il faudrait 63 fois 0 et le dernier (le plus à droite en 1). Sachant cela on sait que l'ordinateur ou d'autre système comme application Web etc va lire cela comme une valeur null si celui-ci utilise base64. 


On devrait mettre en base64 classique == cela permet de dire que c'est la dernière valeur de notre base64 donc on devrait mettre AA== sur le site JWT.io nan ? 

Eh bien en réalité on est sur une base64 URL, donc pour les == sont "omitted" (ignoré) donc pas besoin on peut mettre juste AA. 

Maintenant que vous avez compris pourquoi j'ai mis AA en signature de mon token JWT nous allons essayer de faire du path traversal en changeant la valeur du header kid dans notre token JWT. 

![](../img/Pasted%20image%2020260418003305.png)

Réponse : 

![](../img/Pasted%20image%2020260418003315.png)

Peut-être que j'ai fais une erreur il cherche un fichier dans le dossier keys/ ? peut-être que si je rajoute encore une fois ../ cela va me remettre au dessus ou est-ce que je suis bloqué dans le dossier keys/  ? 


![](../img/Pasted%20image%2020260418003421.png)
Réponse  : 


![](../img/Pasted%20image%2020260418003436.png)


Okay je suis bloqué dans le dossier, mais pourquoi ? techniquement c'est censé marché. Ah mais attendez, pourquoi il cherche keys/dev/null et pas keys/../../dev/null ? Y'a t'il un filtre comme un replace en python ? Pour cela essayons de doublé tout cela( on aura 4 . et 2 /) .


![](../img/Pasted%20image%2020260418003800.png)

![](../img/Pasted%20image%2020260418003822.png)

Okay parfait, il regarde bien les caractère ../ maintenant, essayons de continuer et de voir combien de fois il faut de return pour que cela marche (je vais juste rajouter un nombre n de ../ pour voir comment il réagit au delà de 7 c'est vraiment abusé haha)


![](../img/Pasted%20image%2020260418003941.png)

![](../img/Pasted%20image%2020260418004005.png)

Encore une fois ? 

![](../img/Pasted%20image%2020260418004035.png)

![](../img/Pasted%20image%2020260418004103.png)

Bingo ! 

Mais explication en vif ? 

Le système de filtre (WAF) ou autre devait enfaite juste remplacer la combinaison "../" par "" pour évité le Path traversal, mais il à fallut juste doublé les caractère, il capte une fois, donc remplace par un caractère null 

exemple 

![](../img/Pasted%20image%2020260418004630.png)

Voilà j'espère que cela vous auras aider :). 