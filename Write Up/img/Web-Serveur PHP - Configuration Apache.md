
Dans ce challenge il faudra qu'on applique du code arbitraire directement sur le serveur pour lire le flag qui se trouve dans /private/flag.txt
![](Pasted%20image%2020260421174058.png)

Nous pouvons voir qu'une ressource nous ai donnée et que celui-ci est la doc du fichier de configuration Apache local et non globale. 

On nous dis comme quoi le serveur à filtrer les fichier .php, on pourrait tenter de faire du null-bytes spoiler alert cela ne marche pas mais essayer tout de même. Car ce n'est pas le but du challenge. 

Le but du challenge est de modifier la configuration local Apache via le fichier .htaccess pour changer la façon dont le serveur traite les fichiers dans un répertoire en particulier. Car oui notre .htaccess s'applique seulement dans le dossier où il se situe. 

Comme je l'ai dis ce n'est pas un fichier de configuration Globale comme apache2.conf mais un fichier de configuration Apache local.

Mais pour l'instant voyons comment ce comporte le site : 

