
Pour ce challenge nous allons devoir contourner le filtrage ip.

Lorsqu'on arrive sur le site web nous arrivons sur une page tel quel : 



![[Pasted image 20260321125156.png]]
Ici nous pouvons voir que je n'appartient pas au LAN du site web 

Bon nous pouvons tenter de nous connecter mais cela ne servira à rien il n'y as pas de compte c'est justement pour faire perdre du temps. 

Nous allons maintenant voir ce qui se passe dans la requête et la réponse normal dans un logiciel nommé Burpsuite : 

Voici notre requête lorsqu'on tente de soumettre un identifiant et un mot de passe : 

![[Pasted image 20260321125818.png]]

C'est une requête très basique mais ici nous ne pouvons pas nous connecter si nous ne faisons pas partie de l'intranet pour cela je vais simplement chercher un Header HTTP qui permet de changer d'ip ou d'en ajouter une ip au "client" qui est nous.


![[Pasted image 20260321125024.png]]
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-For
On clique sur la documentation de mozilla et on apprend comment marche cette header. 
# X-Forwarded-For header

The HTTP **`X-Forwarded-For`** (XFF) [request header](https://developer.mozilla.org/en-US/docs/Glossary/Request_header) is a de-facto standard header for identifying the originating IP address of a client connecting to a web server through a [proxy server](https://developer.mozilla.org/en-US/docs/Glossary/Proxy_server).

A standardized version of this header is the HTTP [`Forwarded`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Forwarded) header, although it's much less frequently used.

Cette Header est un header qui permet d'identifier l'adresse IP du client ce connectant au serveur WEB, ici cela va être notre porte d'entrer puisqu'il nous suffit d'avoir une IP privé. Je rappel les pool d'adresse Privé connus sont les suivant (RFC 1918). 

| Préfixe        | Plage IP                      | Nombre d'adresses  |
| -------------- | ----------------------------- | ------------------ |
| 10.0.0.0/8     | 10.0.0.0 – 10.255.255.255     | 232-8 = 16 777 216 |
| 172.16.0.0/12  | 172.16.0.0 – 172.31.255.255   | 232-12 = 1 048 576 |
| 192.168.0.0/16 | 192.168.0.0 – 192.168.255.255 | 232-16 = 65 536    |

Je vais personnellement utiliser l'adresse IP 192.168.1.1 car j'aime bien et c'est tout MDR. 

Bon maintenant nous allons regarder la syntaxe de notre Header HTTP: 

`X-Forwarded-For: <client>, <proxy>`
`X-Forwarded-For: <client>, <proxy>, …, <proxyN>`

Voici l'exemple ci-dessous : 
`X-Forwarded-For: 2001:db8:85a3:8d3:1319:8a2e:370:7348 
`X-Forwarded-For: 203.0.113.195` 
`X-Forwarded-For: 203.0.113.195, 2001:db8:85a3:8d3:1319:8a2e:370:7348`
Première exemple notre client à une adresse IPV6 qui est détecter. 
Deuxième exemple une adresse IPV4 (celle qu'on va utiliser pour notre adresse Privé, se référencer au RFC 1918).

Et enfin l'adresse IPV4 de notre client qui passe par un proxy qui lui à une adresse IPV6, si le client passe par plusieurs proxy, le proxy le plus à droite sera le plus récent (celui qui à fait la requête) et celui le plus à gauche celui qui est au plus proches du clients. 

Bon cela ne nous intéresse pas les proxys pour l'instant mais c'est important de comprendre comment cela marche. Une image très simple de IONOS va me permettre d'économiser un grand temps : 

![[Pasted image 20260321131012.png]]

Bon retournons à notre exploit (qui ne vas pas utiliser de proxy)

Je vais donc rajouter dans mon Header HTTP l'option : 

X-Forwarded-For : 192.168.1.1

Voici à quoi ressemble ma requête : 
![[Pasted image 20260321131236.png]]

Vous pouvez mettre l'option où vous voulez tant qu'il est au dessus de Connection car on utilise ici un Header et non du JSON si c'était du JSON on aurait mis en dessous de Connection: 

![[Pasted image 20260321131359.png]]

Vous pouvez le voir aussi dans Render : 
![[Pasted image 20260321131435.png]]

Bingo vous venez de réussir le challenge mais aussi de faire votre première Spoofing ici vous avez fait de l'ip spoofing (usurpation d'adresse IP)