Tout d'abord regardons l'énoncer de ce challenge : 

![Pasted image 20260405210812.png](Pasted%20image%2020260405210812.png)

Maintenant que nous savons que ça un rapport à notre ancien challenge API nous savons à peut près les manipulations qui ont été faite et pensez aussi au réglage qui ont été fait mais voyons voir plus en profondeur. 


![Pasted image 20260405210913.png](Pasted%20image%2020260405210913.png)

Nous arrivons sur une page qui nous présente l'API de root-me (c'est juste pour le CTF pas le vrai haha)

Maintenant essayons de voir qu'elle sont les différentes options de cette API. 

Nous avons deux requête POST possible qui se situe dans /api/signup et api/login voyons voir le format du request et qu'est-ce que demande la request en terme de donné JSON. 

![Pasted image 20260405211050.png](Pasted%20image%2020260405211050.png)

Okay il suffit de mettre un username et un password pour pouvoir créé un compte pour l'instant tout paraît assez logique. 

![Pasted image 20260405211123.png](Pasted%20image%2020260405211123.png)

Deux réponse possible de notre API, réponse avec le code 201 qui nous affirme que notre utilisateur à été créé. Mais aussi le code 400 qui dis comme quoi l'utilisateur existe déjà (Je suppose qu'il se base juste sur le username.)

Bon continuons voyons voir pour le login maintenant : 

![Pasted image 20260405211530.png](Pasted%20image%2020260405211530.png)

La même chose cette fois ci, le code 200 nous dis juste si ont à réussi à se login, et le code 400 qu'on à un problème dans le username ou le password que l'ont à saisie. 

Bon voyons voir d'autre chose. 

![Pasted image 20260405211648.png](Pasted%20image%2020260405211648.png)

Tiens intéressant, l'option /api/user nous permet de connaître (Grâce à la requête GET) les informations de notre compte. Peut-être qu'il y aura une faille avec ça on verra plus tard pour l'instant continuons. Si nous sommes bien connecter sur le compte l'api nous retournera les différentes information suivante : 

`{`
	`"id" : notre ID`
	 `"username" : "notre nom d'utilisateur"`
	 `"note" : "les différentes notes qu'on à pu mettre avec notre utilisateur"`
	 `"status" : "si nous somme un visiteur ou un admin je suppose : "`
`}`

Voyons voir maintenant l'option api/note qui nous permet de poster une note sur notre profil. 


 
![Pasted image 20260405212044.png](Pasted%20image%2020260405212044.png)

Ici on peut voir qu'elle utilise la requête PUT. Qui je rappel pour ceux qui ne sont habitué que d'utiliser POST ou GET, nous permet de remplacer une donné, ici c'est un strings. Ce qui veut dire que si ma note de départ été "test" et qu'après je refait une requête avec le PUT avec la valeur "Gozen le bg" nous n'aurons pas comme avec le POST "test" ,"Gozen le bg" mais plûtot cela écrasera l'ancienne donnée et mettra juste dans les notes de l'utilisateur : "Gozen le bg" (En vrai c'est vrai que je suis plûtot bg ) Bref continuons, cette requête nous intéresseras je vous invite de la gardé dans un coin de votre tête. Le code 200 dis que notre note à bien été publié et 400 quand nous ne somme pas connecté à un compte.


Et enfin l'api qui contiendra notre Flag qui est /api/flag. 
![Pasted image 20260405212610.png](Pasted%20image%2020260405212610.png)

Celui-ci nous renvois   200 "Hello admin, here is the flag : {le flag root me}" 

Celui-ci nous renvois 401 "Unauthorized, user is not admin"

Il suffit qu'on soit admin haha. 




Bon commençons par créé un compte et voir ce que cela nous donne.

(Remarque les différentes manipulation je l'ai ferais via le site mais en réalité j'ai utiliser Burpsuite pour mieux voir et récupérer le cookie de ma session etc pour faire des tests que je vous montrerais mais je vais vous montrer côté du site pour que ce soit plus compréhensible.)

Bon nous allons créé notre compte : 
Allez sur l'api /api/signup et cliquez sur "Try it out"

Mettez les informations que vous voulez pour vos compte pour ma part : 

![Pasted image 20260405214707.png](Pasted%20image%2020260405214707.png)

Cliquez sur la cellule execute : 

![Pasted image 20260405214819.png](Pasted%20image%2020260405214819.png)

Okay il à bien été crée. Mais attend est-ce qu'il existe un utilisateur nommée admin ? Nous allons voir cela grâce à l'erreur que dois nous renvoyez l'API : 

![Pasted image 20260405214943.png](Pasted%20image%2020260405214943.png)

Okay parfait on pourra plus tard essayer de faire quelques chose avec cela, pour l'instant connectons nous à notre utilisateur pour ma part "username" : "Gozen2", "password " : "Gozen2"



![Pasted image 20260405215109.png](Pasted%20image%2020260405215109.png)

Toujours pareil nous pouvons voir que j'ai mis les données en JSON de mon username et password (j'ai fait sur burpsuite mais vous pouvez voir qu'on doit mettre aussi en JSON de toute façon vous remplacez juste les champs sur le site rien de bien compliquez haha). 

Bon continuons essayons de poster une note sur notre profil. 

Rendez vous dans /api/note et mettez une note : 
![Pasted image 20260405215346.png](Pasted%20image%2020260405215346.png)

Okay la note c'est bien mise, maintenant allons voir l'option /api/user qui permet de voir les informations relative à notre utilisateur, ici c'est Gozen2. 

![Pasted image 20260405215550.png](Pasted%20image%2020260405215550.png)



Nous pouvons voir que j'ai le statut "guest" (visiteur) comme je le pensais au départ. Mais aussi ma note qui est apparût on peut voir que j'ai l'id 3 et qu'il y a bien mon username.


Mais où peut-être la faille dans tout cela ? 

En réalité il faut faire des tests. Si vous êtes ici c'est pour apprendre vos mieux tester et faires des erreurs mais pour vous faire gagner du temps je vais vous dire mes deux Hypothèse la première qui n'était pas bonne, et la deuxième qui été la bonne.


1er Hypothèse : "Si je prend mon cookie et que je trouve les informations relative à l'utilisateur admin et que je trouve la forme du cookie et sont secret de sont cookie grâce à un crackage de secret cookie je suis censé récupérer le bon cookie et donc me co ?"

Oui j'ai tenter cela. Bien trop compliquez pour rien puisque vous allez voir la bonne réponse. Mais je vais vous montrer ce que j'ai tenter. 

Tout d'abord j'ai lancer mon exegol. Par la suite j'ai remarqué que le cookie été un cookie flask. Il me faut juste la forme du cookie pour après tenter de le reconstitué et trouver le secret(salt) relié à ce cookie. 

Faisons cela : 

![Pasted image 20260405220231.png](Pasted%20image%2020260405220231.png)
J'utilise l'outil flask-unsign qui permet de décortiqué un cookie, comme vous pouvez le voir ici le cookie de session est sous forme 

 `{'_fresh': True, '_id': '4f423b9974c473002c0079eafeb06bd5e22ea7d6589c156613a91869ae39976fb6e07abac24bc9e7ab823e36513038ca7b5da82a6f776b130807ccb2e59e16c7', '_user_id': '3'}`

Bon j'ai tenter de crack grâce à l'option --wordlist mais comment vous dire cela n'as pas marché. J'ai donc décidé de me dire que le secret devais être vide. J'ai donc essayer cela : 
![Pasted image 20260405220434.png](Pasted%20image%2020260405220434.png)
(C'est un ancien screen mais j'ai mis 0 en id car c'est souvent celui de l'admin ou d'un bot admin)

Mais bref cela n'as pas marché aussi c'est normal ce n'est pas comme ça qu'il faut faire. 

Maintenant la vraie façon (La 2 ème hypothèse) : 


Je me suis dis, puisqu'il y a une méthode PUT, il est peut-être possible que d'autre option n'ont pas été filtrer par méthode et qu'on peut changer des informations ? 


Pour cela, c'est tout simple il suffit de prendre l'api /api/user qui permet de récupérer des données d'utilisateurs grâce à la méthode GET. 

Par la suite mettez les informations que vous voulez relativement à votre compte comme dans le screen ci dessous, vous pouvez voir que dans le côté "Request" j'ai pris mon cookie de session que j'ai récupérer au préalable lorsque j'ai cliquez sur "signup" avec Burpsuite mais vous pouvez les récupérer en faisant CTRL + SHIFT + I => Application => Cookie 

Et vous prenez le cookie qui à pour nom "session".
Le reste ne se qu'on vos cookie personnel de google analytics. 

Maintenant que vous avez récupérer allez dans Burpsuite et rajouté : 

Cookie: session= "Votre cookie que vous venez de récupérer" 

Faite comme ci dessous : 

![Pasted image 20260405210605.png](Pasted%20image%2020260405210605.png)

Vous pouvez voir que j'ai rajouté en JSON les données de mon api/user mais que j'ai juste changé les différentes données, je me suis dis que l'admin ne devais pas avoir de note donc "" , statut j'ai changer "guest" en "admin" mon userid de 3 à 0 et mon username de "Gozen2" à "admin". Mais ce qui est le plus important c'est le status pourquoi ? 

Car c'est ce qui permet de savoir votre rôle, êtes-vous un visiteur ? ou un admin ? 

À vous d'en décidez grâce à cette donnée JSON haha. 

Vous pouvez voir dans la réponse que nous avons reçu un message "User updated successfully", sûrement que le dev utiliser cette même requête s'il doit changer les données d'un utilisateurs mais nous on à capter cela haha. Mais où est la faille ? nous avons juste mis les données d'un soit disans User nan ? 

Oui vous avez raison. En réalité c'est deux chose combiné qui donne la faille. Tout d'abord l'étape des données relié à notre compte (cookie) car oui, si vous faite cela sans changé la méthode de la requête, dans la réponse vous aurez que les données reliée à votre compte, regardé je fais le test pour vous : 

![Pasted image 20260405221610.png](Pasted%20image%2020260405221610.png)

Bon maintenant regardé bien ce que je vais surlignée : 

![Pasted image 20260405221658.png](Pasted%20image%2020260405221658.png)
Tiens tiens tiens, c'est pas notre méthode PUT ? et oui en réalité j'ai supprimé les données de mon compte et remplacé par ce que j'ai mis, maintenant je suis admin haha. Récupérons notre Flag grâce au /api/flag. 



![Pasted image 20260405221809.png](Pasted%20image%2020260405221809.png)

Bingo ! On se revoit pour un autre chall :)




