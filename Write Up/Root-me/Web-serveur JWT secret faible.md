
![](../img/Pasted%20image%2020260415222618.png)

Dans ce challenge il nous faudra craquer le secret (signature) du token JWT.

Comme déjà expliquez dans un précédent challenge, le token JWT détient trois champs qui sont les suivants : 

- Header 
- Payload
- Signature

C'est trois champs sont séparer par des "." il y a au total 3 "." 

Lorsqu'un JWT est créé celui-ci détient plusieurs paramètre à prendre en compte. Pour le header ce sont les suivantes : 

`{`

  `"typ": "JWT",`

  `"alg": "HS512"`

`}`

Le type ici notre token est un JWT (JSON Web Token)
Et l'alg est l'algorithme qui est utilisé pour encoder les données. 

Pour le payload plusieurs choses peuvent être configuré au bon vouloir de l'administrateur, tel que le rôle du token etc. Exemple dans notre cas avec le token qu'on aura (je vous montrerais comment faire pour récupérer). 

`{`

  `"role": "guest"`

`}`

Maintenant que l'ont connais cela il manque un truc nan ? 

Pour la signature c'est un peu plus spéciale. C'est ce qui permet de rendre ce token un peu plus sécurisé. Car oui il y a une signature pour les gens normaux comme guest par exemple mais aussi pour les admins.

Si celui-ci est trouver, c'est gagner pour l'attaquant. Car il pourra se faire passé par un administrateur ou mettre sont token en temps qu'admin en encodant les différents paramètre comme en changeant le rôle "guest" en "admin" et en mettant le secret (signature) trouver et en le réencodant. 

C'est ce que nous allons faire ici. Maintenant que nous avons pris amplement connaissance du challenge je vous invites à voir ce qu'on peut faire maintenant et à prendre connaissance du site qui nous ai donnée.

![](../img/Pasted%20image%2020260415223436.png)

Voici à quoi ressemble le site web de départ. Il nous ai dis qu'il souhaite que l'ont participe à un jeu et qu'on trouve le secret admin (signature) et qu'on utilise le token pour avoir accès au /admin en utilisant la méthode HTTP : "POST"
Autrement dis : Il faut réussir à ce faire passé pour un administrateur et pouvoir faire une requête "POST" valide auprès du serveur.

Pour ce faire récupérons le token de base qui nous ai donnée dans le /token:

![](../img/Pasted%20image%2020260415223657.png)

Voici le token qu'il souhaite que l'ont utilise : 
`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJyb2xlIjoiZ3Vlc3QifQ.4kBPNf7Y6BrtP-Y3A-vQXPY9jAh_d0E6L4IUjL65CvmEjgdTZyr2ag-TM-glH6EYKGgO3dBYbhblaPQsbeClcw`

Mais comment craquer la signature ? 

Comme beaucoup le savent il y a beaucoup manière de craquer un mot de passe. Enfin quand je veux dire beaucoup de manière je veux dire de tools. Mais ici je vais vous présenté ce que j'ai utiliser. 

Je suis allez voir directement si hashcat prend en compte les tokens JWT. Pour cela je vous invite d'allez sur : 

https://hashcat.net/wiki/doku.php?id=example_hashes

Faite CTRL + F et mettez JWT dans la barre de recherche. 

![](../img/Pasted%20image%2020260415223949.png)

Bingo la méthode est la numéro 16500 (la méthode est le numéro que l'ont va mettre pour dire à hashcat de quel forme est notre hash. Ici on mettra donc -m 16500)

![](../img/Pasted%20image%2020260415224103.png)

utiliser echo et mettez votre token dans un fichier. Ici je créé un fichier qui s'appelle JWT-Token.txt dans lequel il contiendra notre hashes.

Maintenant passons à la commande finale : 

![](../img/Pasted%20image%2020260415224206.png)

Laisser moi vous expliquez -m permet de dire à hashcat qu'elle méthode nous allons utiliser, nous avons vu que c'est la méthode 16500. L'option -a permet de dire qu'elle type de combinaisons de dictionnaire on veut faire, ici l'option 0 sera amplement suffisant. Elle permet de ne comparer que ce qu'il y a dans le dictionnaire avec le hash, car si on met d'autre option il peut faire des combinaisons en mélangeant les MDP etc pour trouver le meilleur mais on ne veut pas faire cela ici. Par la suite on met notre fichier qui contient notre hash. 
Voyez cela comme un sac dans lequel il contient notre hash et ont donne à la hashcat pour qu'il comprend qui est ça cible. Et on lui donne le dictionnaire ici j'utilise rockyou.txt qui est bien connu et suffisant. 

L'option --potfile-disable permet de dire à hashcat que je souhaite craquer le mot de passe même si je l'ai déjà trouver auparavant. Car j'ai retaper la commande pour vous montrer en direct sinon il me met un message comme : "Le hash à déjà été trouver taper la commande --show pour le voir"
Sauf qu'ici je veux recommencer comme vous, vous ne devez donc pas le mettre. 

Maintenant voyons voir ce qu'il nous à trouver: 

![](../img/Pasted%20image%2020260415224654.png)

Bingo le secret est "lol" vous pouvez le voir après le dernier ":" à la fin de notre hash. 

Mais comment on fait maintenant pour le mettre dans notre token ? 

Pour cela je vous invite d'allez sur le site https://www.jwt.io
![](../img/Pasted%20image%2020260415224844.png)

![](../img/Pasted%20image%2020260415224931.png)

Dans la partie de gauche vous allez rentrer le hash que l'ont à craquer. 

NB : En orange c'est le "header" , en violet le "payload" et en vert la "signature"

Vous pouvez voir à droite qu'il à trouver le Header, le payload mais pas encore le secret car c'est à nous de le mettre mais avant cela je vous invite à allez dans la rubrique JWT Encoder : 

![](../img/Pasted%20image%2020260415225009.png)

Vous voyez maintenant que le côté à changer et que cela ce que l'ont met à gauche cela nous génère un token JWT. 
![](../img/Pasted%20image%2020260415225025.png)


Essayons déjà de changer le rôle en admin : 
![](../img/Pasted%20image%2020260415225131.png)

Okay on voit que le payload (la partie en violet) à bel est bien changer comparer à l'ancien(regarder bien la fin n'as pas changer mais c'est vers le milieu / fin)

Maintenant nous allons changer cette signature :
![](../img/Pasted%20image%2020260415225400.png)

qui est celle par défaut de JWT avec le mot de passe que nous venons de craquer : 
![](../img/Pasted%20image%2020260415225513.png)

![](../img/Pasted%20image%2020260415225525.png)

Voici notre token final : 

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJyb2xlIjoiYWRtaW4ifQ.y9GHxQbH70x_S8F_VPAjra_S-nQ9MsRnuvwWFGoIyKXKk8xCcMpYljN190KcV1qV6qLFTNrvg4Gwyv29OCjAWA`

Je vous invite maintenant à prendre une requête vous pouvez utiliser ce que vous voulez tel que curl ou même Burpsuite, dans mon cas je vais utiliser BurpSuite. 

![](../img/Pasted%20image%2020260415225636.png)
Allez dans Proxy puis Intercept et vous le mettez en On, Par la suite ouvrez leur navigateur via le bouton "open browser". 

Dès que cela est fait taper l'url admin sur le navigateur : 
http://challenge01.root-me.org/web-serveur/ch59/admin

Cela va vous mettre en réponse "The method is not allowed for the requested URL."
Mais cela est normal puisqu'on essaye d'avoir une page via la méthode GET alors qu'on veut déposer une information mais cela c'est juste pour récupérer cela : 
![](../img/Pasted%20image%2020260415225820.png)

Maintenant cliquez dessus et faite CTRL + R cela va renvoyer directement à l'onglet Repeater qui va nous permettre de modifier notre requête : 

![](../img/Pasted%20image%2020260415225949.png)

Mettez la méthode en POST en haut à gauche tout d'abord : 

![](../img/Pasted%20image%2020260415230013.png)
Comme vous pouvez le voir ici en haut j'ai mis la requête post, une erreur nous ai renvoyez il faut qu'on mette le Header Authorization : Bearer + Le token qu'on à fait sur le site JWT.io 

On aurait pu s'en douter car je ne vous l'ai pas dis. Pour importé un JWT il faut mettre le Header  HTTP Authorization, mais dans cette header plusieurs option sont possible, donc pour savoir que c'est un JWT le paramètre est Bearer. 

Et on met enfin notre token JWT craquer ce qui donne : 
![](../img/Pasted%20image%2020260415230231.png)

Voici ce que j'ai rajouter je l'ai surligner. Testons pour voir : 

![](../img/Pasted%20image%2020260415230334.png)
Voici notre flag : ATTENTION le "\n" ne fait pas partie du flag. 

![](../img/Pasted%20image%2020260415230450.png)

Bingo !! 