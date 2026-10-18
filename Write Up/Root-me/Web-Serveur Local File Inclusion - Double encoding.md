
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

Lien pour les fichiers communs en PHP si besoin : 

https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/Programming-Language-Specific/Common-PHP-Filenames.txt



Nous pouvons remarquer que lorsque nous venons d'utiliser l'intruder de Burpsuite, une requête n'a pas la même taille que les autres et qu'il n'y a pas de contenu ou qu'il n'est pas affiché.

![](../img/Pasted%20image%2020261018201914.png)

Nous allons donc nous pencher vers ce fichier.