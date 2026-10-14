

Tout d'abord, nous allons regarder ce qui nous est demandé dans le challenge.


![](../img/Pasted%20image%2020261014175101.png)


Ok, notre but ici est de nous rendre dans la section administrateur.

Maintenant, voyons voir comment se présente le site. 

![](../img/Pasted%20image%2020261014175218.png)

Le site a pour titre "File viewer", ce qui veut dire que nous pouvons voir le contenu ou les fichiers qui sont stockés dans le serveur du site.

On peut remarquer qu'il y a un bouton pour la connexion en administrateur. Malheureusement nous n'avons aucun "credentials" sous la main pour se connecter. 


Voyons voir les sections, naturellement je me tournerais vers la section sysadm qui est la section pour les fichiers d'administration système et donc un possible script pour la connexion vers la section administrateur.


![](../img/Pasted%20image%2020261014175510.png)

Okay, on peut remarquer qu'il y a un paramètre dans l'URL avec le nom de la variable qui est file et qui a pour valeur sysadm.

![](../img/Pasted%20image%2020261014182057.png)

Nous pouvons remarquer que lorsqu'on essaye d'accéder au code de la page index.html, nous avons dans l'URL un nouveau paramètre qui s'appelle f et qui a pour valeur le nom du fichier.

Maintenant que nous savons cela, nous allons essayer d'effectuer ce qu'on appelle un Path Traversal, comme on a pu déjà faire dans un autre challenge en Web-client sur lequel je n'ai pas encore fait de write-up car le principe est plutôt simple et je vais vous l'expliquer.

Le principe du Path Traversal est plutôt simpliste, lorsque vous êtes sur un système Unix ou même Windows, nous savons qu'il est possible de remonter dans les dossiers parent ou même les fichiers qui se situent dans un dossier parent.

Pour ce faire, il suffit de taper les caractères ".." qui permettent de remonter dans les dossiers parents, par exemple dans un terminal Linux ou Windows, on effectue la commande : 

```bash
C:\Users\****> cd ..
```


Le principe est le même en web, si nous mettons dans l'url le caractère ".." cela va nous remonter dans les dossiers parents, 

Essayons de le faire, mais avant tout cela je dois vous prévenir de quelques choses. 

Il est possible de bloquer cette façon de faire du path traversal car il est vraiment classique. Pour ce faire il suffit de faire un script qui parcours chacun des caractères encoder ou normaux et de voir si le doublons du caractères encoder ou même normal se répète un nombre de fois, par exemple deux fois. Sauf que la viens une autre faille.

Si votre script est mal fait, il se peut qu'il ne supprime que les deux caractères ".." et donc si on met "...." cela va supprimer deux fois ".." et donc cela va toujours donner ".." et donc faire un Path traversal.

Pareil pour l'enchaînement "../", on peut faire un script, puis si on fait "....//", cela peut quand même passer si votre script de filtre est mal fait. Attention, il faut vérifier aussi les différents types d'encodage de ces différents caractères.

Bon, revenons à notre challenge.

![](../img/Pasted%20image%2020261014183403.png)

Si nous essayons de le faire dans le champ du deuxième paramètre, nous pouvons voir que cela nous a fait remonter qu'une seule fois et qu'il y a une erreur qui apparaît. 

Essayons maintenant dans le paramètre ?files : 

![](../img/Pasted%20image%2020261014183648.png)

On remarque que nous avons remonté d'un dossier parent par rapport au type catégorie qui sont les dossiers.

Essayons de remonter encore une fois.

![](../img/Pasted%20image%2020261014183817.png)


Bingo, maintenant nous voyons qu'il y a "admin". Essayons de rentrer à l'intérieur : 

![](../img/Pasted%20image%2020261014183908.png)

Okay, nous pouvons voir que nous sommes actuellement dans le dossier admin et qu'il y a un fichier index.php.

Essayons de voir ce que contient le fichier index.php : 
![](../img/Pasted%20image%2020261014184036.png)


Nous pouvons voir qu'il y a une table (array) qui a pour nom de variable "users", essayons de nous connecter avec le nom et le mot de passe qui sont dans le fichier index.php : 


![](../img/Pasted%20image%2020261014184650.png)

