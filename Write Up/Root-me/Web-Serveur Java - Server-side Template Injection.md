
Tout d'abord, prenons en compte ce qui est dit dans l'énoncé.



![](../img/Pasted%20image%2020261015193725.png)


Ici, nous pouvons voir qu'il faut récupérer le flag qui se situe dans SECRET_FLAG.txt.


Grâce au titre du challenge Java Server-side Template Injection, nous savons qu'il faut effectuer une SSTI.

Voyons voir à quoi ressemble le site web. 

![](../img/Pasted%20image%2020261015195439.png)


Nous pouvons voir une entrée utilisateurs, nous allons utiliser le repo de PayloadsAllTheThings pour aiguiller la faille. Voici le lien du repo : 

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection

Voici le payload qui nous permet de savoir si la faille est bel et bien présente.

```java
${{<%[%'"}}%\.
```

S'il y a une faille, nous sommes censés avoir une erreur qui est retournée.

Mais tout d'abord, voyons voir ce qui se passe lorsque je mets mon pseudo : 

![](../img/Pasted%20image%2020261015200514.png)


Okay, mon pseudo est affiché maintenant, tentons le payload, nous sommes censés avoir une erreur qui nous est retournée.

![](../img/Pasted%20image%2020261015200718.png)


Ici, nous utilisons un payload polyglotte qui nous permet en une fois de savoir qu'il y a une faille.

Maintenant que nous avons fait une petite phase de reconnaissance, on va aiguiller cette phase de reconnaissance.

![](../img/Pasted%20image%2020261015200831.png)


Nous allons maintenant aiguiller nos recherches en tentant les payloads, si le test est réussi on suit la flèche verte sinon on suit la rouge.

![](../img/Pasted%20image%2020261015202027.png)

On voit que le calcul s'est effectué. Ce qui veut dire qu'il est possible d'exécuter une commande arbitraire. Maintenant nous allons encore aiguiller en tentant le payload : 

```java
a{*comment*}b
```

![](../img/Pasted%20image%2020261015202139.png)
Cela n'a pas marché, nous savons donc que c'est un moteur en dehors de ceux qui sont ici. Heureusement que dans l'énoncé il est marqué que c'est du java.


Voyons voir la partie Java de PayloadAllTheThings : 

![](../img/Pasted%20image%2020261015202252.png)


On a vu que le payload a marché avec ${ }


Donc, on va essayer Freemarker pour voir si c'est le moteur Freemarker.

![](../img/Pasted%20image%2020261015202608.png)

Ici, on voit que cela a marché aussi, mais on ne sait pas encore si c'est SpEL. Donc, on va essayer différents payloads. 



