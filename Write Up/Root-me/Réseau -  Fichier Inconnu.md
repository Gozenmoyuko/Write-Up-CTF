


Celui-ci va être plutôt rapide et facile à comprendre, voyons voir l'énoncer : 

![](../img/Pasted%20image%2020260405232347.png)

Maintenant que nous savons cela installons le fichier. 

Quand cela est fait, rendez-vous sur une VM linux, sur votre terminal Linux si vous êtes déjà sous linux ou WSL.


Pour trouver le nom de notre téléphone c'est tout simple, nous allons utiliser la commande strings : 

![](../img/Pasted%20image%2020260405232634.png)

Maintenant que vous avez cela vous pouvais voir que le fichier est sous forme btsnoop, mais aussi que le nom du Téléphone est GT-S7390G. 


Maintenant comment savoir pour l'adresse MAC ? C'est simple wireshark prend en charge les fichiers .bin, 

Lançons le fichier sur wireshark. 

![](../img/Pasted%20image%2020260405232840.png)

Dès la première requête nous pouvons trouver l'adresse MAC en allant sur Bluetooth HCI event - Connect Request, car c'est le PDU qui demande la connexion, donc il y a forcément les informations correspondant à la machine qui demande. 

Rappel L'ADRESSE MAC CE MES EN MAJ 

Rendez-vous sur : 

https://gchq.github.io/CyberChef

Mettez l'option SHA1 : 
![](../img/Pasted%20image%2020260405233516.png)

Mettez votre adresse MAC en majuscule et collé au nom de votre téléphone


![](../img/Pasted%20image%2020260405233456.png)

Et bingo ! 
![](../img/Pasted%20image%2020260405233556.png)
