
Dans ce ctf nous allons devoir trouver une configuration qui à été mal faite lors du développement de l'intranet via Nginx.

![Pasted image 20260331204735.png](../img/Pasted%20image%2020260331204735.png)

Maintenant  que nous savons essayons de voir le site : 

![Pasted image 20260331204857.png](../img/Pasted%20image%2020260331204857.png)

Si vous avez l'oeil vous devriez voir qu'il y a une note laisser par le développeur qui dis ce qui aurait du être fait donc Patch un bug essayons d'avoir accès à ce dossier. 

![Pasted image 20260331204950.png](../img/Pasted%20image%2020260331204950.png)

Okay nous voyons que nous avons accès que au retour mais peut-être qu'il y a des fichiers caché et c'est cela que nous allons voir. 

Allons voir le site portswigger (Burpsuite) pour voir qu'est-ce qui peuvent nous dire sur le path traversal de nginx . 


https://portswigger.net/bappstore/a5fdd2cdffa6410eb530de5a4c294d3a

Sur ce lien nous pouvons voir qu'il est expliquer comme quoi les caractères .. peuvent-être considérer tel que : 

https://example.com/static../

devient pour le serveur : 

https://example.com/

ce qui veut dire que selon quelques manipulations utiliser par Nginx nous pouvons faire du Path traversal comme sur un autre ctf. 

Voici les différentes technique et directories et technique qu'utilise Nginx.

Pour une URL tel que  https://example.com/folder1/folder2/static/main.css cela peut générer les liens suivants avec nginx pour :  

https://example.com/folder1../folder1/folder2/static/main.css https://example.com/folder1../%s/folder2/static/main.css 
https://example.com/folder1/folder2../folder2/static/main.css https://example.com/folder1/folder2../%s/static/main.css https://example.com/folder1/folder2/static../static/main.css https://example.com/folder1/folder2/static../%s/main.css

bon essayons dans notre cas :


![Pasted image 20260331205941.png](../img/Pasted%20image%2020260331205941.png)


Cela n'as pas l'air de réussir avec le %s essayons de voir si c'est les .. ou le %s qui bloque pour cela on fait : 

![Pasted image 20260331205958.png](../img/Pasted%20image%2020260331205958.png)

Okay ce n'est pas les .. qui pose problème maintenant essayons de les remettre et d'enlever le %s. 

![Pasted image 20260331210032.png](../img/Pasted%20image%2020260331210032.png)

Parfait cela nous à fait remonté dans le fichier juste au dessus comme nous pouvons le voir, Nginx à pris comme si nous venons de faire : 

 http://challenge01.root-me.org:59092/assets/..

Or ce n'est pas le cas nous avons juste collé donc on est revenu sur le dossier qui hébergé /assets comme nous pouvons le voir on peut y retourné de dans. 

Mais on voit surtout que nous avons notre flag.txt 

Voyons voir si ce n'est pas une fausse piste. 

![Pasted image 20260331210307.png](../img/Pasted%20image%2020260331210307.png)

Bingo !! 


