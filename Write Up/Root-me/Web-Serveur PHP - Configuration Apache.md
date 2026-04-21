
Dans ce challenge il faudra qu'on applique du code arbitraire directement sur le serveur pour lire le flag qui se trouve dans /private/flag.txt

![](../img/Pasted%20image%2020260421174058.png)

Nous pouvons voir qu'une ressource nous ai donnée et que celui-ci est la doc du fichier de configuration Apache local et non globale. 

On nous dis comme quoi le serveur à filtrer les fichier .php, on pourrait tenter de faire du null-bytes spoiler alert cela ne marche pas mais essayer tout de même. Car ce n'est pas le but du challenge. 

Le but du challenge est de modifier la configuration local Apache via le fichier .htaccess pour changer la façon dont le serveur traite les fichiers dans un répertoire en particulier. Car oui notre .htaccess s'applique seulement dans le dossier où il se situe. 

Comme je l'ai dis ce n'est pas un fichier de configuration Globale comme apache2.conf mais un fichier de configuration Apache local.

Nous aurons besoin de cela  : 
https://httpd.apache.org/docs/2.2/fr/howto/htaccess.html

Mais pour l'instant voyons comment ce comporte le site : 

![](../img/Pasted%20image%2020260421175407.png)


Tentons de voir si nous pouvons upload un fichier .php tout simplement. 


![](../img/Pasted%20image%2020260421181106.png)

Bon nous pouvons voir que ce n'est pas autorisé comme il été dis. Tentons un simple null-byte par exemple notre fichier ici qui s'appeler null-bytes.php deviendra en réalité : 

null-bytes.php%00.png

Ici notre fichier est vu tel qu'un fichier png mais en réalité c'est un fichier .php c'est le pricipe du nullbyte qui veut dire "bit/octet null" et qui supprime tout ce qu'il y a après. 

Bon maintenant essayons. 

![](../img/Pasted%20image%2020260421181514.png)

On peut voir que mon null-byte à été accepter, on à bel est bien bypass mais qu'est-ce que cela fait si je clique sur le bouton qui va me rediriger vers mon fichier ? 

![](../img/Pasted%20image%2020260421181615.png)

Tiens le serveur n'interprête pas le null-byte, il l'interprête tel qu'un caractère normal. Donc comme je vous l'avez dis les null byte ne vons pas marché.

Essayons d'upload un .htaccess 

Pourquoi on va utiliser le.htaccess ? Déjà car c'est dans la description du challenge en bleu. 

Mais aussi car comme il nous ai dis sur le lien : 

https://httpd.apache.org/docs/2.2/fr/howto/htaccess.html

**`.htaccess`** est un fichier de configuration **local** utilisé par **Apache HTTP Server**, qui permet de modifier le comportement du serveur **sans accès administrateur (root)**, mais uniquement **dans les limites autorisées par la configuration globale** (apache2.conf, etc... ).

Maintenant que nous savons cela, nous allons pouvoir essayer d'upload un fichier .htaccess

![](../img/Pasted%20image%2020260421182021.png)
Pour l'instant nous allons rien mettre dans le .htaccess et nous allons voir si ont à une réponse http 200 ou pas. Je vous invite à quand vous allez mettre votre fichier .htaccess de récupérer la requête http via Burpsuite par exemple.


![](../img/Pasted%20image%2020260421182146.png)

Okay parfait, nous voyons qu'il à bien été upload, nous voyons ici la réponse: 

Réponse : 
![](../img/Pasted%20image%2020260421182553.png)

On voit que notre fichier ce trouve dans un dossier qui se nomme.

uploads/mn5f66i9fcr0bnja2u7anvlnpj/.htaccess
Rappelez vous, notre fichier .htaccess permet de modifier le comportement d'un dossier mais aussi des sous dossiers du dossier parent. 

Ici nous allons utiliser cela. 

Tout d'abord essayons d'upload un fichier .png l'ambda de test : 

![](../img/Pasted%20image%2020260421182732.png)

Ici c'est un faut PNG je n'ai modifier que le filename de ma requête, ce qui veut dire que sur la requête de .htaccess c'était : 

filename=".htaccess" 
Maintenant c'est : 
filename="test2.png" 

Vous pouvez tout les deux le voir sur la ligne 17. 

Maintenant que cela est fait nous allons mettre un script dans notre test2.png avant le champs "WebkitFormBondary ...." qui est la ligne de fin des donnée de notre fichier directement upload. 
Mais est-ce qu'il se situe dans le même fichier nos mon .png et mon .htaccess sinon c'est plus compliquer ? 
![](../img/Pasted%20image%2020260421183020.png)
Bingo ils sont au même endroit si vous comparer les deux. On va pouvoir modifier notre .htaccess maintenant mais avant essayons de mettre un petit script php dans mon fichier .png pour pouvoir lister les fichiers du répertoire et voir que cela marche quand on aura mis la bonne configuration. 

![](../img/Pasted%20image%2020260421183200.png)

Voici la nouvelle requête. Explication : 

en php nous pouvons exécuté des commandes shell via plusieurs façon ici j'utilise system qui permet directement d'exécuté mais aussi de renvoyer la valeur de la commande. 

Modifions notre .htaccess maintenant en se basant pour la plus part de la configuration sur : 

https://httpd.apache.org/docs/2.2/fr/howto/htaccess.html

![](../img/Pasted%20image%2020260421183701.png)

Nous allons nous intéressé à cette partie. 


Ici on nous dis qu'un fichier .htaccess peut être utiliser pour un répertoire en particulier et que pour cela ils faut mettre des 
