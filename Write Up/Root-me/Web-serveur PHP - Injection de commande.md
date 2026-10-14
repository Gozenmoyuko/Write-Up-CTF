
Tout d'abord, prenons en compte l'énoncé : 

![](../img/Pasted%20image%2020261015002801.png)

Okay, parfait maintenant nous savons qu'il faut réussir à lire le fichier index.php.


Voyons voir à quoi ressemble le site web : 

![](../img/Pasted%20image%2020261015002905.png)

Rien de très étonnant, comme expliqué dans l'énoncé, c'est un site qui permet d'envoyer des paquets ICMP (Ping). 

Si le ping est réussi, cela veut dire que le serveur a reçu un paquet ICMP response. 

Bon, là n'est pas vraiment le sujet. Ce qui nous intéresse est d'exécuter du code arbitraire sur la machine. 

Maintenant, si on réfléchit un peu, nous savons que le serveur exécute la commande ping de son côté. 

Nous avons donc deux possibilités pour effectuer une injection de commande. Soit on fait en sorte d'exécuter des instructions en plus en passant par la façon PHP ou même en ligne de commande, donc en fermant via le caractère ";"
Soit on fait un pipe "|" pour effectuer une commande à la suite du "ping". 

(Spoiler alert les deux marches)


Bon, tentons tout d'abord : 

```bash
127.0.0.1;ls -la
```

![](../img/Pasted%20image%2020261015003959.png)

Bingo ! 

Maintenant essayons l'autre option : 

![](../img/Pasted%20image%2020261015004035.png)

Bingo aussi ! 

Seule différence est que le pipe renvoie seulement la dernière commande, ce qui est normal. L'utilisation du pipe est normalement pour utiliser la sortie d'une commande dans une autre, comme par exemple cat /etc/passwd | grep -e "root" 

Bon, maintenant rien de très compliquer, il nous ai demander de regarder le fichier index.php donc on va faire un : 
```bash
127.0.0.1;cat index.php
```
ou 
```bash
127.0.0.1|cat index.php
```


Bon, en réalité je ne trouve pas cela très pratique car le code de la page réapparaît et qu'il faut faire des magouilles. 

Si vous êtes un habitué, vous savez donc directement que les flags sont dans .passwd très souvent dans root-me. 

Je vais effectuer donc : 

```bash
127.0.0.1;cat .passwd
```

Mais avant cela, je vous montre ce qui se serait passé si on avait fait : 

```bash
127.0.0.1|cat index.php
```


![](../img/Pasted%20image%2020261015004609.png)

Comme vous pouvez le voir, ce n'est pas ouf... 

Donc, je vais effectuer : 

```bash
127.0.0.1;cat .passwd
```

![](../img/Pasted%20image%2020261015004929.png)

Bingo, vous venez d'avoir le flag, essayons de valider : 
![](../img/Pasted%20image%2020261015005008.png)

Bien joué ! Rendez-vous dans un autre challenge ! 
