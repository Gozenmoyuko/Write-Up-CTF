
Tout d'abord voyons voir notre but : 

![](../../img/Pasted%20image%2020260516123417.png)

Énoncé : 

Lancez-vous dans la « Dimensional Escape Quest », où vous vous réveillez dans un mystérieux labyrinthe forestier qui semble venir d’un autre monde. Frayez-vous un chemin parmi des écureuils chantants, des nymphes espiègles et des sorciers grincheux dans un labyrinthe fantaisiste qui pourrait vous réserver des surprises d’un autre monde. Parviendrez-vous à venir à bout de ce labyrinthe enchanté ou vous perdrez-vous dans une autre dimension pleine de défis magiques ? Le voyage commence dans cette évasion mystique !


Tout d'abord voyons voir comment ce comporte l'application et à quoi elle ressemble.

![](../../img/Pasted%20image%2020260516123749.png)

Okay ça à l'air d'un petit jeu sympa essayons le ! : 

![](../../img/Pasted%20image%2020260516123816.png)

4 options s'offre à nous.

Personnellement je prends le NORD car il faut pas perdre le nord de notre mission :) 

![](../../img/Pasted%20image%2020260516123902.png)

Okay c'est bon maintenant je vais suivre le chemin mystérieux parce que je suis un aventurier hehe. 

![](../../img/Pasted%20image%2020260516123945.png)

Je vais tenter la carte de l'aventurier mais peut-être que cela me conduira vers ma perte ? 

![](../../img/Pasted%20image%2020260516124104.png)

Tiens tiens bizarre choisissons une autre option. 
![](../../img/Pasted%20image%2020260516124145.png)

![](../../img/Pasted%20image%2020260516124223.png)

Finalement il fallait faire un camp bon essayons de rentrer dans le portail magique.

![](../../img/Pasted%20image%2020260516124307.png)

Bon j'ai perdu et ça ma saouler essayons de voir ce qu'il faut faire.


Tout d'abord nous pouvons voir dans le main que plusieurs fichiers sont disponible : 

![](../../img/Pasted%20image%2020260516124437.png)

Essayons d'ouvrir les 3 dans notre navigateur.


Voici le contenu du commands.js : 

![](../../img/Pasted%20image%2020260516124534.png)

Nous pouvons voir qu'il export une variable type constante donc non modifiable qui s'appelle GAME_WON mais aussi GAME_LOST.

Maintenant essayons de voir le contenu de main.js : 

![](../../img/Pasted%20image%2020260516124712.png)

Bon nous pouvons voir que le fichier est assez long.

On peut notamment voir que la fonction playerWon et playerLost sont importé depuis game.js, Essayons de voir s'il y a playerWon dans le fichier main.js.

![](../../img/Pasted%20image%2020260516125316.png)

On peut voir que nos options qui ont marché sont juste ici, mais bizarre il n'y en as que 3 alors que ça doit ce finir après le SET UP CAMP. Donc techniquement avec toutes les options nous pouvons pas finir le jeu.

On voit qu'il essaye de récupérer des informations d'un dossier /api essayons de voir s'il n'y as pas par exemple un code de l'api qui traine qui à toute les options à l'intérieur.


![](../../img/Pasted%20image%2020260516125508.png)

Bingo il n'y en as que deux donc on à notre /api/monitor et /api/options.

Regardons le premier.

![](../../img/Pasted%20image%2020260516125607.png)

Bon rien de très intéressant. Regardons /api/options.

![](../../img/Pasted%20image%2020260516125638.png)

Okay bingo rejouons au jeu et partons de cette aventur ! Ici on peut voir qu'il y a plusieurs commandes. Or on connais déjà les 3 premières commandes qui sont les bonnes qui je rappel sont : 

HEAD NORTH
FOLLOW A MYSTERIOUS PATH
SET UP CAMP
et pour le 4ème on à vu que rien ne marché. Essayons au quatrième de mettre la phrase du secret qui est : 

Blip-blop, in a pickle with a hiccup! Shmiggity-shmack


![](../../img/Pasted%20image%2020260516130022.png)

BINGO !! 

![](../../img/Pasted%20image%2020260516130047.png)
:) 

