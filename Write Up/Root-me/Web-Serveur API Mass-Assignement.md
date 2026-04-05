Tout d'abord regardons l'énoncer de ce challenge : 

![[Pasted image 20260405210812.png]]

Maintenant que nous savons que ça un rapport à notre ancien challenge API nous savons à peut près les manipulations qui ont été faite et pensez aussi au réglage qui ont été fait mais voyons voir plus en profondeur. 


![[Pasted image 20260405210913.png]]

Nous arrivons sur une page qui nous présente l'API de root-me (c'est juste pour le CTF pas le vrai haha)

Maintenant essayons de voir qu'elle sont les différentes options de cette API. 

Nous avons deux requête POST possible qui se situe dans /api/signup et api/login voyons voir le format du request et qu'est-ce que demande la request en terme de donné JSON. 

![[Pasted image 20260405211050.png]]

Okay il suffit de mettre un username et un password pour pouvoir créé un compte pour l'instant tout paraît assez logique. 

![[Pasted image 20260405211123.png]]

Deux réponse possible de notre API, réponse avec le code 201 qui nous affirme que notre utilisateur à été créé. Mais aussi le code 400 qui dis comme quoi l'utilisateur existe déjà (Je suppose qu'il se base juste sur le username.)

Bon continuons voyons voir pour le login maintenant : 

![[Pasted image 20260405211530.png]]

La même chose cette fois ci, le code 200 nous dis juste si ont à réussi à se login, et le code 400 qu'on à un problème dans le username ou le password que l'ont à saisie. 

Bon voyons voir d'autre chose. 

![[Pasted image 20260405211648.png]]

Tiens intéressant, l'option /api/user nous permet de connaître (Grâce à la requête GET) les informations de notre compte. Peut-être qu'il y aura une faille avec ça on verra plus tard pour l'instant continuons. Si nous sommes bien connecter sur le compte l'api nous retournera les différentes information suivante : 

`{`
	`"id" : notre ID`
	 `"username" : "notre nom d'utilisateur"`
	 `"note" : "les différentes notes qu'on à pu mettre avec notre utilisateur"`
	 `"status" : "si nous somme un visiteur ou un admin je suppose : "`
`}`

Voyons voir maintenant l'option api/note qui nous permet de poster une note sur notre profil. 


 
![[Pasted image 20260405212044.png]]

Ici on peut voir qu'elle utilise la requête PUT. Qui je rappel pour ceux qui ne sont habitué que d'utiliser POST ou GET, nous permet de remplacer une donné, ici c'est un strings. Ce qui veut dire que si ma note de départ été "test" et qu'après je refait une requête avec le PUT avec la valeur "Gozen le bg" nous n'aurons pas comme avec le POST "test" ,"Gozen le bg" mais plûtot cela écrasera l'ancienne donnée et mettra juste dans les notes de l'utilisateur : "Gozen le bg" (En vrai c'est vrai que je suis plûtot bg ) Bref continuons, cette requête nous intéresseras je vous invite de la gardé dans un coin de votre tête. Le code 200 dis que notre note à bien été publié et 400 quand nous ne somme pas connecté à un compte.


Et enfin l'api qui contiendra notre Flag qui est /api/flag. 
![[Pasted image 20260405212610.png]]

Celui-ci nous renvois   200 "Hello admin, here is the flag : {le flag root me}" 

Celui-ci nous renvois 401 "Unauthorized, user is not admin"

Il suffit qu'on soit admin haha. 





Commençons par créé un compte et voir ce que cela nous donne.


![[Pasted image 20260405210605.png]]






