
Pour cela on peut savoir déjà que nous allons manier de la mémoire ou soit récupérer un fichier mémoire par le mot "dump".


Commençons en examinant tout d'abord l'énoncer. 

![[Write Up/img/Pasted image 20260307163026.png]]


Il nous ai expliquer que le site sur lequel nous allons travailler est un site de blog dans lequel il y a donc des blogs (logique MDR) mais aussi un système de manipulation avec une API et notre mission est de trouver le fichier dans lequel il y a le flag. 


Bon lançon le challenge.


![[Write Up/img/Pasted image 20260307163402.png]]

On arrive bel est bien sur un site de blog dans lequel si vous tentez on peut presque ne rien faire, pas de réelle endpoint à priorie. Attendez que vois-je ? un tag avec une documentation sur les APIs, cliquons dessus et essayons de voir. 

![[Write Up/img/Pasted image 20260307163531.png]]

Testons de regarder un peu les options possible sur le site et de trouver un endpoint, une faille, ou une fonction pour récupérer le flag. 

Cliquons sur la premières option. 

![[Write Up/img/Pasted image 20260307163729.png]]

tiens le lien parait suspect. on peut naviguez grâce à cette fonctions de l'API nous pouvons essayer de voir si lorsqu'on marque get_flag si cela va récupérer un element flag ou quelque chose d'autre.


![[Write Up/img/Pasted image 20260307163838.png]]

Bon malheureusement il ne se passe rien, je pense personnellement qu'on ne pourras rien récupérer avec les options de l'API dans le get puisque lorsque j'éxécute la suite du lien avec plusieurs donnés il n'y à aucun chargement de page ou autre. 

Voyons voir la prochaine catégorie. 


![[Write Up/img/Pasted image 20260307164007.png]]

Bon je ne vois pas trop de solutions qui puisse m'intéressais  on voit qu'on à deux fonctions get mais j'ai tenter pour vous rien d'intéressant.


![[Write Up/img/Pasted image 20260307164531.png]]


Tiens tiens tiens. Ce n'est pas le titre de notre challenge mais aussi ce qui été marquer dans l'énoncer ? le fait qu'il fallait lire la mémoire. Exécutons toute suite l'option de l'api. 



![[Write Up/img/Pasted image 20260307164710.png]]


Plusieurs options s'offre à vous pour récupérer le fichier. Soit vous récupérer le lien de heapdump dans le curl, ou soit vous cliquez sur Download file. Puisque c'est accès rare d'avoir un bouton download explicitement je ne l'avais pas vu j'ai fais par lien. J'ai vérifier et ce sont les mêmes fichiers.

![[Write Up/img/Pasted image 20260307164839.png]]

![[Write Up/img/Pasted image 20260307164855.png]]
voici les deux fichiers on peut voir qu'ils ont pas le même point bizarre. Mais j'ai regarder le flag ce trouve dans les deux. Mais ça explique ce que je disais juste avant, il est possible que le file qui peut s'installer est tout simplement retraiter avant d'être upload donc si vous travaillez sur un vrai site ou autre faite attention à cela toujours avoir le bon réflexe de prendre le fichier à la source directement. 


![[Write Up/img/Pasted image 20260307165247.png]]


Voici ce qu'il y a dans le fichier. Bon vous inquiétez pas vous n'aurez pas à avoir tout d'obfusquer ou lire la mémoire nous savons que le flag commence par pico{ alors faisons un simple CTRL + F et cherchons le flag.

![[Write Up/img/Pasted image 20260307165433.png]]

Bingo vous pouvez le mettre et récupérer vos points. 

