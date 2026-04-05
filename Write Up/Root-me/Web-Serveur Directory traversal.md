
Pour ce challenge nous n'auront pas forcément besoin d'avoir de compétence en informatique, bien que plusieurs méthode sont possible pour trouver ce que l'ont cherche par le biais de plusieurs Header html. Je vous présente la manière la plus simple et par la suite nous verrons avec la manipulation de Header que l'ont peut directement s'informé sur la documentation de mozilla. 


Bon voyons voir tout d'abord l'énoncer. 

![Write Up/img/Pasted image 20260321215737.png](Write%20Up/img/Pasted%20image%2020260321215737.png)


Il faut qu'on trouve une section qui est cachée dans la galerie photo, pour mieux comprendre comment bien navigué et trouver des endpoints etc je vous conseilles de vous référer à tout ce qui est mis en ressource. Avec cela vous pouvez facilement finir le challenge. 

Bon voici la page sur laquelle on apparaît : 

![Write Up/img/Pasted image 20260321220610.png](Write%20Up/img/Pasted%20image%2020260321220610.png)


lorsqu'on cliques sur une catégorie nous pouvons voir que nous avons un paramètre dans l'url qui se nomme galerie et qui à pour valeur l'onglet sur lequel vous avez cliquez pour ma part j'ai cliquez sur apps j'ai donc : 

![Write Up/img/Pasted image 20260321220807.png](Write%20Up/img/Pasted%20image%2020260321220807.png)

Bon maintenant tentons de regardé la racine des fichiers grâce au parametre pour cela on va juste mettre après le = un /
![Write Up/img/Pasted image 20260321220853.png](Write%20Up/img/Pasted%20image%2020260321220853.png)

Bon on peut voir qu'il y a nos différentes catégorie mais qu'une est apparue. qui s'appelle 86hwnX2r
Pour pas vous faire chier à le recopier aller dans le code source de la page et chercher la valeur et copier collé la. 

Bon maintenant qu'on as cela on peut simplement faire la chose suivante qui est mettre comme valeurs de paramètre ce qu'on viens de trouver : 

![Write Up/img/Pasted image 20260321221118.png](Write%20Up/img/Pasted%20image%2020260321221118.png)

Jackpot maintenant allons voir le code source de la page pour voir où ce situe le fichier password. 

![Write Up/img/Pasted image 20260321221305.png](Write%20Up/img/Pasted%20image%2020260321221305.png)

Bon maintenant changons simplement l'url par cela : 

![Write Up/img/Pasted image 20260321221400.png](Write%20Up/img/Pasted%20image%2020260321221400.png)

Et voilà le tour est jouer

![Write Up/img/Pasted image 20260321221433.png](Write%20Up/img/Pasted%20image%2020260321221433.png)

Bingo ! 

Maintenant il y a une deuxième façon de faire plus compliquez pour trouver le fichier grâce à : 

Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: file

