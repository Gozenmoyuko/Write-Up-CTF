
Dans ce challenge nous allons devoir montrer que nous  pouvons avoir un accès arbitraire à la machine et lire des fichiers sensible tel que /etc/passwd mais ici nous allons pas directement manipuler celui-ci mais devoir trouver les paramètres de la machine et de la configuration  du serveur. 

Pour cela nous allons devoir fouillez et regardé où peut ce situé la faille Path traversal. 

Pour la faire simple selon plusieurs doc que j'ai pu lire et des rapports d'une tel faille, la configuration de location Nginx qui est sensible à cette faille sont celle qui ne finissent pas par "/" 
ce qui veut dire là où le location ce fini par un fichier. 

Mais comment cela marche ? 

Tout simplement ici nous pouvons voir que la root est configuré dans /etc/nginx. 
Le fichier dis en autres que pour les différentes location comme "/" cela nous retourne directement dans /login/login.html 

Mais si on part de l'inverse qu'est-ce que ça fait ? 
En réalité rien si on met simplement /login/login.html dans l'URL. 

Mais qu'est-ce que ça fais si on essaye de retourné en arrière et de voir la configuration de nginx ? le fichier nginx.conf qui je le rappel est le fichier par défaut de tout serveur nginx. 
https://nginx.org/en/docs/beginners_guide.html

![img/Pasted image 20260402205205.png](img/Pasted%20image%2020260402205205.png)


Maintenant que nous savons cela tentons de voir la réponse du serveur lorsqu'on essaye de parler à ce lien. 

Je rappel on part de /login/login.html on remonte de deux arboresence pour retomber sur le / et par la suite on essaye de lire le fichier par défaut de nginx qui est nginx.conf ce qui donne. 

/login/login.html/../../nginx.conf

![img/Pasted image 20260402204537.png](img/Pasted%20image%2020260402204537.png)

Okay nous pouvons voir ici que le serveur nginx inclus un fichier default.conf qui se trouve dans le dossier conf.d tentons d'y avoir accès et de le lire : 

![img/Pasted image 20260402221637.png](img/Pasted%20image%2020260402221637.png)

On retombe sur le script de notre énoncer descendons dans la requête : 

 ![img/Pasted image 20260402221743.png](img/Pasted%20image%2020260402221743.png)

Bingo vous pouvez valider le challenge maintenant ! 

