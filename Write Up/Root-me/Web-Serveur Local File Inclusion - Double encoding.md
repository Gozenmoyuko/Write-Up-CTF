
Tout d'abord, nous allons prendre connaissance de ce qui nous est dit dans l'énoncé.


![](../img/Pasted%20image%2020261018194505.png)

Si vous vous souvenez du challenge que nous avons fait dans le même thème qui sont les LFI, vous saurez que le principe est simple, par exemple depuis un paramètre d'URL nous pouvons simplement accéder à un autre fichier.

Maintenant, le problème est que dans ce cas-là, il est explicitement dit grâce au titre du challenge qu'il va falloir que l'on effectue du double encodage.

Gardons cette idée en tête et commençons par voir comment nous pouvons exploiter le challenge.

![](../img/Pasted%20image%2020261018195323.png)

Voici à quoi ressemble le site, lorsque nous arrivons à l'intérieur.

Essayons maintenant de nous y balader.

![](../img/Pasted%20image%2020261018195441.png)

Comme nous pouvons le voir, lorsque j'ai cliqué sur le bouton CV, l'URL a changé en rajoutant un paramètre qui s'appelle page avec pour valeur CV.

Je vous rappelle que le "?" Est le préfixe pour un paramètre dans l'URL et que le "=" permet d'affecter une valeur.

Maintenant, essayons de mettre une valeur un peu au hasard.


![](../img/Pasted%20image%2020261018195759.png)

Nous pouvons voir qu'une erreur apparaît, actuellement j'ai affecté la valeur "s" et voici ce que cela m'a retourné. Maintenant, regardons bien en détail ce que nous avons reçu. Nous pouvons remarquer qu'il essaye de visiter un fichier s'appelant "s.inc.php", bizarre non ? On dirait la valeur que j'ai mise en paramètre. 

On remarque aussi qu'il cherche dans le chemin /usr/share/php, sauf qu'il n'y arrive pas car le fichier "s.inc.php" n'existe pas dans ce répertoire.

Maintenant, essayons de faire du Path traversal. Cela consiste à remonter dans le répertoire parent une ou plusieurs fois en utilisant "../" pour pouvoir accéder depuis l'endroit où nous sommes à un fichier généralement restreint.

![](../img/Pasted%20image%2020261018200250.png)

Mince, il détecte notre attaque. Mais rappelez-vous de ce que nous avons lu au début du challenge. On se rappelle que le nom du challenge contenait "Double-encoding" ce qui explique le bypass du filtre (qui doit être au passage un WAF très certainement) qui filtre un simple encodage ou même les caractères d'accéder ou de remonter pour éviter l'attaque de Path Traversal.

Pour encoder, je vais actuellement utiliser le décodeur de burpsuite qui peut permettre aussi d'encoder.

![](../img/Pasted%20image%2020261018210320.png)

Essayons d'encoder une fois en percent encode(URL encode) pour voir si nous sommes restreints de remonter dans les fichiers, par exemple si nous sommes bloqués dans un même dossier ou non.




![](../img/Pasted%20image%2020261018215647.png)

Comme nous pouvons le voir, l'attaque a été détectée.

Maintenant, si nous lisons quelques informations qui nous sont données dans les PDF, il nous est dit que si nous mettons %25, cela peut dépasser les WAF, car celui-ci représente le caractère "%" qui peut être lui-même bloqué. Or, si nous passons par la valeur %25 qui est interprétée comme un "%", alors nous pouvons contourner. Ce qui revient à un double encodage URL.


Ce qui donne : 
```html
/index.php?page=%252e%252e%252f
```

Maintenant, essayons de voir ce qu'il se passe lorsqu'on essaye de remonter d'un dossier parent. 

![](../img/Pasted%20image%2020261018224112.png)

Comme nous pouvons le voir, le système n'est pas autorisé à remonter d'un dossier. Il va donc falloir que nous trouvions un autre moyen de pouvoir récupérer le fichier index.php.

On va donc essayer maintenant de trouver un fichier sensible sur le site qui aurait été oublié et qui a pour extension .inc.php qui est un "include php". On peut retrouver quelques listes de noms de fichier avec ce type d'extension et qui sont des fichiers dis commun. Ce qui veut dire qu'il y a beaucoup de fichiers qui se nomment ainsi.

Lien pour les fichiers communs en PHP si besoin : 

https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/Programming-Language-Specific/Common-PHP-Filenames.txt

Maintenant, je vais utiliser l'intruder de burpsuite en récupérant une requête. 

![](../img/Pasted%20image%2020261018224548.png)

Comme vous pouvez le voir, j'ai mis dans le paramètre page notre indicateur pour Burpsuite pour lui dire que c'est ici qu'il va devoir tester les différentes requêtes en modifiant par les valeurs dans une liste.

Maintenant, à droite je clique sur "Add from list" et je sélectionne la liste qui se nomme "Filenames long"
 
Nous pouvons remarquer que lorsque nous venons d'utiliser l'intruder de Burpsuite, une requête n'a pas la même taille que les autres et qu'il n'y a pas de contenu ou qu'il n'est pas affiché.

![](../img/Pasted%20image%2020261018201914.png)

Nous allons donc nous pencher vers ce fichier.

Ce que je vais faire maintenant, c'est de prendre le chemin pour le fichier et d'utiliser une option de PHP pour pouvoir lire les données non visibles en les encodant en base 64 pour voir si on récupère quelque chose ou si ce n'est qu'un faux positif.

Pour ce faire, nous allons utiliser : 

```php
php://filter/convert.base64-encode/resource=conf
```

Explication : Ici nous allons utiliser PHP pour récupérer les données que nous avons filtrées, pour les encoder nous utiliser convert.[nom_type_conversion]. Par la suite, on met le fichier source qu'on pense qu'il y a une donnée cachée qui est "conf".


Maintenant essayons ! 

![](../img/Pasted%20image%2020261018225518.png)

Mince, pourquoi cela n'a pas marché ? Car nous avons oublié qu'il faut encoder nos caractères. 
En repassant par burpsuite voici les encodages qu'on a besoin.

```
: => %3a
/ => %2f
. => %2e
```

Maintenant, n'oubliez pas qu'il faut que l'on rajoute pour la double encodage URL le %25 devant ce qui donne :

```
: => %253a
/ => %252f
. => %252e
```

Voici le lien final : 



```php
php%253a%252f%252ffilter%252fconvert%252ebase64-encode%252fresource=conf
```

Essayons le ! 


![](../img/Pasted%20image%2020261018225750.png)

Bingo ! Essayons de voir si la théorie était bonne ! 

![](../img/Pasted%20image%2020261018225844.png)

Bien joué ! Vous venez de terminer le challenge on se revoit bientôt ! 