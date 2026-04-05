

Pour ce challenge nous arrivons vers un site plutôt simple dans lequel on vous demande simplement de créé un compte et de vous y connecté. 

Par la suite il faut vous créé un compte et vous y connecté. Vous pouvez voir plusieurs pages disponible tel que : 
![Write Up/img/Pasted image 20260321115538.png](Write%20Up/img/Pasted%20image%2020260321115538.png)
Lors de votre investigation vous pouvez voir que contact permet d'envoyer un mail vers un administrateur pour faire une demande. Profil est tout simplement votre nom et votre statut. Ce sera important pour la suite : 
![Write Up/img/Pasted image 20260321115629.png](Write%20Up/img/Pasted%20image%2020260321115629.png)
et enfin vous avez Private un onglet réserver aux administrateur dans lequel il faut aller chercher le flag. 

Bon revenons tout simplement à notre challenge: 
Vous pouvez voir que le status est tout simplement grisé et vous comprenez vite que seul l'administrateur du site peux coché cette case pour passer quelqu'un d'autre administrateur sur le site. 

Vous allez me dire qu'elle est le rapport avec le Token CSRF ? Tout d'abord voyons le fonctionnement grâce à une image simple à comprendre de Vaadata. 

![Write Up/img/Pasted image 20260321120102.png](Write%20Up/img/Pasted%20image%2020260321120102.png)

Donc c'est un token qui est transmit à l'utilisateurs lors de sa visite du site. 

Bon ici malheureusement nous allons pas manipuler le token ou récupérer le token d'un administrateur c'est bien plus simple c'est juste qu'on doit faire en sorte que l'administrateurs coche la case status et clique sur submit. 

Bon voyons voir le code de la page maintenant. 

![Write Up/img/Pasted image 20260321120435.png](Write%20Up/img/Pasted%20image%2020260321120435.png)

Pour trouver le code j'ai simplement cliquez sur CTRL + Shift + I et j'ai cliquez sur l'icon en haut à gauche pour me renvoyer sur le code ou ma souris ce situe j'ai cliquez là où il y avait le formulaire. 

Nous pouvons remarquer qu'ici nous avons une requête post qui se fait, ce qui paraît plutôt logique petite pique de rappel des différentes requête.

![Write Up/img/Pasted image 20260321120842.png](Write%20Up/img/Pasted%20image%2020260321120842.png)
Si vous êtes familier vous pouvez ne pas regarder sinon c'est bien de les connaître, si vous avez des difficultés en anglais faites l'effort d'essayer de comprendre et s'il y a un mot que vous ne connaissez pas traduisez le et retourner à votre lecture. Du temps pour vous de "perdu" mais en réalité qui vous feras gagner du temps. 

Bref continuons. 

Si on retourne sur notre pages web on voit que la notre username ce trouve dans "value" ce qui veut dire que c'est grâce à cela qu'on peut différencier plusieurs utilisateurs. On va utilisez tout le form et juste lui ajouter un nom dans lequel nous allons pouvoir faire une requête automatique grâce au javascript. Mais avant cela tentons de faire en sorte de cocher la case juste avec du code html. 

![Write Up/img/Pasted image 20260321121347.png](Write%20Up/img/Pasted%20image%2020260321121347.png)

Ici ce trouve notre bouton "checkbox", ici le nom est directement relier au statut et on voit qu'il est en disabled. Essayons de laisser pour l'instant disabled et voir si on peut cocher. Vous allez voir que c'est simple, puisque c'est une checkbox html nous avons juste à rajouter checked= qui permet de dire si boutons doit être de base en coché ou décocher. Dans notre cas on va le mettre en coché donc mettez à la suite du disabled checked="checked"
![Write Up/img/Pasted image 20260321121656.png](Write%20Up/img/Pasted%20image%2020260321121656.png)

Bingo la case est bien coché maintenant revenons à l'endroit où il est possible de faire notre exploit qui est l'onglet "Contact". 

![Write Up/img/Pasted image 20260321121827.png](Write%20Up/img/Pasted%20image%2020260321121827.png)

Pour cela rien de très compliqué, il n'y as pas de filtre au niveau de l'email vous pouvez mettre test@gmail cela marchera. 

Maintenant on pourra faire notre failles sur la sections comment. Pour cela nous avons juste à copier le code de la requête html "post" qui ressemble à ceci : 
`<form action="?action=profile" method="post" enctype="multipart/form-data">`
		`<div class="form-group">`
		`<label>Username:</label>`
		`<input type="text" name="username" value="testa">`
		`</div>`
		`<br>`		
		`<div class="form-group">`
		`<label>Status:</label>`
		`<input type="checkbox" name="status" disabled="">`
		`</div>`
		`<br>`	
		`<button type="submit">Submit</button>`
		`</form>`

Bon maintenant qu'on à cela je vais supprimé tout ce qui n'est pas forcément intéressant tel que les balises div, label, br etc... 
`<form action="?action=profile" method="post" enctype="multipart/form-data">`
		`<input type="text" name="username" value="testa">`
		`<input type="checkbox" name="status" disabled="">`
		`<button type="submit">Submit</button>`
		`</form>`


Je n'ai gardé que ce qui est important pour notre exploit. (les inputs qui contient notre valeurs utilisateurs, la checkbox pour nous coché et le bouton submit qui va nous permettre d'accepter la modification.)

Bon maintenant nous allons ajouté à nom à notre formulaire pour le manipuler et faire en sorte d'envoyer le document (la requête POST) via le bouton submit. 
<`form name="csrf" action="http://challenge01.root-me.org/web-client/ch22/index.php?action=profile" method="post" enctype="multipart/form-data">``
    ``<input type="text" name="username" value="testa" />``
    ``<input type="checkbox" name="status" checked=checked />``
    ``<input type="submit" value="Submit request" />``
`</form>`

Okay ici vous pouvez voir que je n'ai qu'ajouter un nom à notre formulaire, maintenant envoyons le grâce a la balise script. 

`<form name="csrf" action="http://challenge01.root-me.org/web-client/ch22/index.php?action=profile" method="post" enctype="multipart/form-data">`
    `<input type="text" name="username" value="testa" />`
    `<input type="checkbox" name="status" checked=checked />`
    `<input type="submit" value="Submit request" />`
`</form>`
`<script>document.csrf.submit()</script>`

Ici, j’ajoute l’attribut `name="csrf"` à mon formulaire afin de pouvoir y accéder facilement en JavaScript via `document.csrf`.  
Ensuite, j’utilise la méthode `.submit()` pour forcer l’envoi automatique du formulaire, sans interaction de l’utilisateur.

Maintenant rendons nous dans l'onglet private.

![Write Up/img/Pasted image 20260321123510.png](Write%20Up/img/Pasted%20image%2020260321123510.png)

Bingo !! 