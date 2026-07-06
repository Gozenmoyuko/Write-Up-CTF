![](../../img/Pasted%20image%2020260706172843.png)

Ils nous ai dis que c'est une application Web qui utilise des tokens de types JWT qu'on à déjà pu voir sur différents challenge Rootme. Il faut trouver un secret qui est cacher et il faut qu'on trouve la faille de sécurité.


![](../../img/Pasted%20image%2020260706173109.png)


On peut voir ici que nous avons différents champs dans lequel on peut mettre différentes informations comme notre nom, notre adresse e-mail et une description pour envoyer un ticket.

Tentons d'envoyer un ticket. 

![](../../img/Pasted%20image%2020260706173548.png)


Nous pouvons voir que nous n'avons pas de token et donc pas de session utilisateur ou autre. 

Maintenant voyons voir les requêtes et réponses qui se passent entre le serveur et notre machine grâce à Buprsuite.

![](../../img/Pasted%20image%2020260706173655.png)

On peut voir qu'il n'y as rien qui se passe il n'y as pas de champs tokens. 

Voyons voir le code source de la page. 

![](../../img/Pasted%20image%2020260706174018.png)

Le flag est le secret_key du token JWT. 


## Résoudre

Pour résoudre cette erreur, il faut que le token jwt ne soit pas créer directement dans le code source de la page. Mais aussi que le script ce passe sur le serveur externe. Et que le secret du token soit assez complexe pour ne pas être craquer. Il faut savoir que les tokens JWT on plusieurs champs et que tout les champs peuvent être des vecteurs d'attaque.