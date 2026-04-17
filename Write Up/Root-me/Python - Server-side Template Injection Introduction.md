
![](Write%20Up/img/Pasted%20image%2020260417232025.png)

Tout d'abord il faut savoir comment marche un Server-side Template.

Tout simplement il va récupérer les informations d'un utilisateurs et créé une page .html ou autre avec ses informations, cela peut servir notamment pour faire des dashboard personnaliser ou autre.

Voyons voir comment marche ce site : 

![](Write%20Up/img/Pasted%20image%2020260417232201.png)

On dirais que c'est un site pour créé une page, comme ils nous ai dis dans l'énoncer. Testons de créé une page avec test et test.

![](Write%20Up/img/Pasted%20image%2020260417232250.png)

Okay on peut voir que nos deux informations sont ici. Le deuxième champs est le contenu donc la première information qui apparaît et le titre le deuxième qui apparaît. Cela nous servira vous verrez.

Mais comment savoir qu'elle champs détient une faille SSTI ? 

Tout simplement nous allons tester le payload polyglote bien connu de ce github : 

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/

Juste grâce à ce github vous pouvez déjà réussir le challenge en réfléchissant un peu, je vous laisse faire avant de continuer le challenge. 


Bon testons nos différents champs via ceux payload qui est trouver lorsque vous descendez un peu vers : 

[PayloadsAllTheThings/Server Side Template Injection at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#universal-detection-payloads)

```powershell
${{<%[%'"}}%\.
```

Pour savoir où le serveur exécute quelques choses il faut voir dans qu'elle champs on à une page d'erreur qui apparaît. 


![](Write%20Up/img/Pasted%20image%2020260417232717.png)

Okay ce n'est pas dans le titre (premier champs) peut-être dans le deuxième ? 

![](Write%20Up/img/Pasted%20image%2020260417232751.png)

Bingo ! 

Maintenant, on sait que via le titre du challenge que c'est une faille SSTI du à un service SST basé sous Python.

Mais comment savoir lequel c'est ? 

![](Write%20Up/img/Pasted%20image%2020260417232853.png)

Comme vous le voyez il n'y a que deux catégorie de syntaxe ici. Ceux qui sont régit par ${ } et ceux qui sont régit par {{ }}

Nous allons tester les deux dans notre deuxième champs (content) et voir lequel nous renvoie une page d'erreur. 

![](../img/Pasted%20image%2020260417233152.png)

Okay ce n'est pas le premier, voyons voir si je me suis tromper et que ce n'est pas un qui est répertorier ? Car oui il peut y avoir des personnaliser (moins connue) et donc plus compliquer à trouver la faille. 

![](../img/Pasted%20image%2020260417233253.png)

Bingo c'est le deuxième. 

On sais ici que nous voulons lire le flag car "Exploitez pour lire le flag" dans l'énoncer. Or si on regarde : [PayloadsAllTheThings/Server Side Template Injection/Python.md at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#django)

Django ne peut pas lire de fichier. Ou soit il n'y as pas de faille exploitable sur cela. Si vous êtes habitué vous devez savoir qu'un des plus connu est Jinja2. 

Essayons de voir s'il y a une faille pour lire des fichiers ou soit faire du code arbitraire via Jinja2. 

[PayloadsAllTheThings/Server Side Template Injection/Python.md at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2)

En réalité il vous suffit de faire des copiés collés et de voir lequel marche. Mais testons de faire cela plus intelligemment. Pour vous faire gagner du temps j'ai tester les plusieurs façon pour lire le fichier directement. J'ai donc décidé de passé dans la catégorie du Remote Command Execution [PayloadsAllTheThings/Server Side Template Injection/Python.md at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2---remote-command-execution)

Car grâce à cela nous pouvons faire du code arbitraire sur le serveur directement. 

![](../img/Pasted%20image%2020260417234248.png)
Essayons de voir si on import os et qu'on essaye d'exécuté ID cela marche. 

![](../img/Pasted%20image%2020260417234344.png)

![](../img/Pasted%20image%2020260417234358.png)

Bingo ! essayons maintenant de remplacer id par "ls -la" pour lister tout les fichiers qu'il y a : 

![](../img/Pasted%20image%2020260417234435.png)

![](../img/Pasted%20image%2020260417234459.png)

Okay bingo. Maintenant je vous invite à copier tout cela, le mettre dans un block note et d'essayer de regarder cela tranquillement.

On voit un petit fichier 4 lignes en partant du haut .passwd essayons de le lire via "cat". 

![](../img/Pasted%20image%2020260417234637.png)

Bingo !!

