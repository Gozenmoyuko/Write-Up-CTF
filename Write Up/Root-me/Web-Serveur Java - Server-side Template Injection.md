
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


Nous allons maintenant aiguiller 