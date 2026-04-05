
![[Pasted image 20260402233313.png]]


Dans ce challenge il va falloir trouver le cookie flask de notre utilisateur admin mais aussi devoir trouver le mot de passe secret utilisé par notre administrateurs et qui est stocké sur le serveurs grâce à l'outil flask-unsign. 

Pour cela tout d'abord prenons en compte l'environnement de travail sur lequel on est : 

![[Pasted image 20260402233803.png]]

Nous arrivons sur cette page, essayons d'accéder au panel admin : 

![[Pasted image 20260402233949.png]]

Bon nous ne sommes pas un utilisateurs admin pour cela essayons de récupérer notre cookie flask qui nous ai désigner en arrivant sur le site. 

![[Pasted image 20260402234128.png]]

Bon voici mon cookie flask : 

eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9.ac7XoA.vvOPhPs3GMiM-hqpKr--HrTgtBA

Bon maintenant comme je vous l'ai dis on va utiliser flask-unsign mais tout d'abord essayons de voir comment est la forme des paramètres du cookie flask. 

Prenons ce site : https://gchq.github.io/CyberChef/ 

![[Pasted image 20260402234622.png]]


Bon maintenant copions ce qui nous à été donné et changons le paramètre admin qui est en "false" en "true"


gardons la bonne version : 
{"admin":"true","username":"guest"}iÎ× ï8øO³qÈj¤ªÇ­8-
maintenant que nous avons cela, ce que vous voyez derrière qui est bizarre c'est ce qu'on appel un signet(signature) c'est ce que l'administrateurs met pour sécurisé sont cookie et que le serveur sache que c'est un bon cookie. Sans ce signet si vous changez les paramètres admin comme moi la maintenant cela ne marchera jamais puisque vous transmettez un signet qui n'est pas le même signet que celui pour le paramètre "admin":"false" .

Il faut voir ça comme un mot de passe. Vous avez beau mettre "admin":"true" si vous n'avez pas le bon signet cela ne marchera jamais. 

Heureusement il y a des outils pour crack ce signet via rockyou.txt ou d'autre dictionnaire de mot de passe mais ici on est sur un mot de passe faible.

Pour cela nous allons utilisé : 

https://github.com/paradoxis/flask-unsign

Faite  : 

pip install flask-unsign 

Maintenant que c'est fait vous allez tenté de cracker le signet . 

Pour cela nous allons utilisée  l'option -unsign qui va nous permettre de récupérer les informations mais aussi via une autre option de cracker le signet. 

Par la suite il faut spécifier notre cookie que nous avons qui est 

`eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9.ac7XoA.vvOPhPs3GMiM-hqpKr--HrTgtBA`

On utilise l'option -c (cookie) suivie de --wordlist rockyou.txt 

Et enfin pour accepter les mots de passe avec des chiffres car oui de base flask-unsign n'accepte que les strings on va utiliser l'option --no-literal-eval

La commande final est : 
`flask-unsign --unsign -c "eyJhZG1pbiI6ImZhbHNlIiwidXNlcm5hbWUiOiJndWVzdCJ9.ac7XoA.vvOPhPs3GMiM-hqpKr--HrTgtBA" --wordlist rockyou.txt --no-literal-eval`

![[Pasted image 20260402235637.png]]

Ok on a le signet qui est 's3cr3t' maintenant nous allons réencoder notre cookie flask pour cela on reprend nos paramètre modifier `{'admin':'true', 'username': 'guest'}`

Nous allons utiliser l'option -sign pour faire le contraire et reconsitué notre cookie. 

On va rajouté l'option --cookie suivie de notre paramètre modifier etre `''`
ce qui donne pour l'instant : 

`flask-unsign --sign --cookie '{'admin': 'true', 'username' : 'guest' }'`

dernière étape pour finir notre cookie il faut rajouté le secret trouver dans notre commande pour cela nous allons utilisé l'option --secret "notre secret trouver" suivie de --no-literal-eval toujours puisque nous avons des chiffres dans notre secret.

La commande final est celle-ci : 

`flask-unsign --sign  --cookie '{"admin":"true","username":"guest"}' --secret "s3cr3t" --no-literal-eval`

![[Pasted image 20260403000123.png]]

Ok maintenant essayons de mettre dans Burp le cookie qui à été trouver qui est : 
`eyJhZG1pbiI6InRydWUiLCJ1c2VybmFtZSI6Imd1ZXN0In0.ac7nLQ.BweMOtk1pGcY6g1hKI0qqNKGLmc`
![[Pasted image 20260403000230.png]]

Bingo !! 
