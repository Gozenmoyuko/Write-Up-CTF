Dans ce challenges voici ce qui nous ai demandé : 

![img/Pasted image 20260331000639.png](img/Pasted%20image%2020260331000639.png)


Pour ce faire nous allons tout d'abord ouvrir le site concerné et démarrer le challenge. 

![img/Pasted image 20260331000707.png](img/Pasted%20image%2020260331000707.png)

Nous arrivons sur un site sur lequel nous avons un une galerie photo et un endroit pour upload. 

Rappelons il nous suffit de poster un code en PHP pour réussir le challenge mais si vous pouvez faire cela sur un vrai site vous pourrez exécuté du code arbitraire, récupérer la base de données etc... 

Bon continuons, regardons la section Upload. 

![img/Pasted image 20260331000941.png](img/Pasted%20image%2020260331000941.png)

Okay cliquons sur Upload : 

![img/Pasted image 20260331000959.png](img/Pasted%20image%2020260331000959.png)

Nous pouvons Upload que des gif ou JPEG mais aussi PNG. 

Pour cela nous allons devoir bypass et mettre du PHP. En réalité pour ce challenge nous allons pas nous attardé sur le contenu du php puisque ce n'est pas ce que l'ont souhaite. On souhaite juste vous initié à savoir upload et bypass les filtres dédiés au différents fichiers tel qu'ici les images. 


![img/Pasted image 20260331001212.png](img/Pasted%20image%2020260331001212.png)

Voici mon code php, évité de mettre un nom aussi explicite lorsque vous faites des failles haha. Bien-sûr à ne réalisé que sur des sites dans lequel vous êtes autorisé ou protéger par des agents politique (DGSI,DGSE ). Bon maintenant vous allez me dire comment faire ? Si on tape sur internet juste "null bytes" nous pouvons voir plusieurs sources qui combine "path traversal" un autres challenge plutôt simple que je vous conseilles d'aller voir, et justement notre null byte.

Mais qu'est-ce que le null byte ? Comme sont nom l'indique c'est un bit nul, nous savons que souvent les bits étant compliquer à lire nous préférons les représenté sous forme hexadecimal. ça forme en binaire pourrait être 0000 selon la taille de donnée que vous envoyer. (D'autre faille ce base sur l'envoie de beaucoup de 0 ou autre mais ici oublié on est en web). Enfaite on va tromper le navigateur en mettant une fausse extension tel que png, gif ou JPEG dans ce cas.

Mais en réalité ce sera du code php. Comme vous pouvez le voir on s'en fou pour l'instant du code à l'intérieur.  Bon maintenant c'est très simple le null bytes encodé en URL-encoded. Pour cela c'est simple, on sait que en hexadécimal le null byte c'est 0x00 ici c'est simple on sais que les URL-encoded commence par %. 

Donc cela donne %00. 

Mais que faire maintenant ? 

Imaginons votre fichier s'appel upload.php. Vous allez mettre votre null byte encodé en URL directement après le .php ce qui donne 

upload.php%00

Vous allez me dire "mais on à pas encore mis en png ou gif ou que sais-je". 

Et je vous répondrais "oui c'est normal c'est la prochaine étape".

Maintenant vous allez rajouté votre extension, si vous êtes sur Windows ne vous enfaite pas du message d'avertissement de changement d'extension et des données soit disant qui disparaissent.

Cela donne : 

upload.php%00.png

Si vous êtes sur linux au lieu d'un clique droit et faire à la main vous pouvez juste faire : 

mv upload.php upload.php%00.png

et tada maintenant essayons de mettre sur le serveur notre fichier.

![img/Pasted image 20260331002444.png](img/Pasted%20image%2020260331002444.png)

Okay maintenant je vais cliquez sur upload en espérant que c'est bon. 

![img/Pasted image 20260331002526.png](img/Pasted%20image%2020260331002526.png)

Okay parfait maintenant retournons dans upload et essayons de voir si y'a mon fichier. 

![img/Pasted image 20260331002736.png](img/Pasted%20image%2020260331002736.png)

Okay parfait maintenant cliquons sur le fichier ! 

![img/Pasted image 20260331002818.png](img/Pasted%20image%2020260331002818.png)

Bingo , au passage vous pouvez voir que le fichier en haut dans l'url est bien .php donc c'est bon !! 

