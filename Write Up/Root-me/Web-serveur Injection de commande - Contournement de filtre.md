

Tout d'abord, voyons voir ce qui nous est dit dans l'énoncé. 

![](../img/Pasted%20image%2020261014230435.png)

Bon, rappelons-nous de l'ancien challenge, qui est un challenge d'injection de commande en PHP.

Il suffisait de finir la ligne avec le caractère ";" puis d'injecter des commandes comme par exemple : 

```bash
ls -la
```

etc... 

Ici, il nous est précisé qu'il y a une couche de sécurité en plus. 

Cela peut être dû, par exemple, à un filtre du caractère ";" ou autre. C'est ce que nous allons voir actuellement. 


Bon, voyons comment se présente le site. 

![](../img/Pasted%20image%2020261014230937.png)

Pour l'instant, nous pouvons voir comme quoi le site à toujours le même design. Voyons voir maintenant comment il se comporte lorsqu'on effectue un ping depuis celui-ci. 

![](../img/Pasted%20image%2020261014231033.png)

Il n'y a rien à signaler du côté du fonctionnement du ping, le paquet ICMP a bel et bien été transmis à l'adresse visée qui est l'adresse de loopback 127.0.0.1. 

Par contre, nous pouvons voir comme quoi le résultat n'est pas directement affiché sur le site, comparé au premier challenge qui montre directement le nombre de paquets qui ont été transmis et le statut de la commande directement sur le navigateur. 

Retenons une théorie, qui est que si l'affichage ne se fait pas sur la page web directement, il va falloir la rediriger sur un site tel que webhook.site ou même dans le collaborateurs de burpsuite.

Maintenant récupérons la requête via Burpsuite. 

![](../img/Pasted%20image%2020261014231547.png)

Parfait, maintenant voyons voir ce qu'il y a dans la réponse. 

Nous venons d'effectuer un ping simple comme juste avant. On peut remarquer que celui-ci a bel et bien fonctionné car nous avons le message "Ping Ok". 

Nous pouvons remarquer qu'il y a un Header nommé "Vary" et que celui-ci détient une valeur "Accept-Encoding".

Ce qui veut dire que nous avons la possibilité sûrement d'utiliser de l'encodage URL pour pouvoir effectuer du code arbitraire ou autre.


Voyons voir maintenant si le caractère ";" est celui-ci bloqué ou non.

