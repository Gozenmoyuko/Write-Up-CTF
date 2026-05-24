![](../../img/Pasted%20image%2020260516170225.png)

`Scénario du défi`
`Une nouvelle tendance fait son apparition : une application qui génère un nom effrayant pour vous. Les utilisateurs de cette application ont découvert par la suite que leurs vrais noms avaient eux aussi été modifiés comme par magie, ce qui a semé le chaos dans leur vie. Pourriez-vous aider à faire disparaître cette application ?`


Tout d'abord voyons voir ce qu'on à lorsqu'on installe les fichiers du site : 

![](../../img/Pasted%20image%2020260516170521.png)


On peut voir ici que le flag ce trouve à la racine de l'application.

Voyons voir un peu le site comment il se comporte maintenant. 



![](../../img/Pasted%20image%2020260516171006.png)

On arrive sur un site plutôt jolie, on nous dis qu'on peut rentrer un nom et que celui-ci deviendra un nom "spooki" effrayant. Essayons une XSS simple connu comme : 

"><script>alert("test")</script>

![](../../img/Pasted%20image%2020260516171140.png)

Okay parfait. 

Mais je ne vois pas comment on peut faire une RCE (Remote command execution) à part avec exec.

Mais bon voyons voir le code source si nous trouvons quelques choses de plutôt intéressant . 

![](../../img/Pasted%20image%2020260516171459.png)

Essayons de voir le main.py. 

![](../../img/Pasted%20image%2020260516171529.png)


On peut voir que le main.py importe flask et surtout flask_mako.

Mais qu'est-ce que Mako ? Mako est un outil basé sous le langage python (flask aussi) est permet de récupérer des informations sur les utilisateurs dynamiquement et de faire des modifications ou des affichage etc. C'est ce qui permet par exemple d'afficher votre pseudo sur un dashboard ou avoir des points dynamiquement et de vous les affichées. 

Maintenant je vais vous présentez qu'est-ce qu'une SSTI (Server Side Template Injection).

Elle permet tout d'abord de voir et de savoir qu'elle service SST est utiliser. Comme mako, par exemple si nous n'avions pas eu le code sources nous auront pu simplement utiliser un payload de type polyglote pour voir si le site (ou le champs choisi) est vulnérable à une SSTI. 

Pour le payload polyglote vous pouvez prendre le suivant : 
```powershell
${{<%[%'"}}%\.
```

Vous êtes censé avoir une erreur de type 500 Internal Server error.

Si c'est le cas le champs est alors vulnérable et le site aussi. 

Maintenant comment l'exploité ? 

Tout d'abord il faut savoir que les informations au niveaux des payloads que je récupère sont retrouvable sur le github PayloadAllTheThings le lien pour y accéder est le suivant : 

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection

Celui-ci représente la section des SSTI mais beaucoup d'autre domaine et payload sont disponible sur ce github, je vous conseille de l'utiliser pour vos CTF cela vous permettras de gagner du temps. 

Bon on sais qu'on est sur du mako mais comment faire si on n'aurais pas eu cette information ? 
Toujours via le github nous pouvons essayer les différentes façons de reconnaissances avant l'attaque. 

![](../../img/Pasted%20image%2020260516172551.png)

Les lignes vertes sont à suivres si la commande est un succès et les lignes rouges si cela à échoué. Mais pour vous montrez je vais emprunté rapidement un chemin rouge et vous comprendrez rapidement.



Bon tout d'abord essayons le simple ```
${7*7}```

![](../../img/Pasted%20image%2020260516172636.png)

Okay cela à marché. On peut voir que le site à interpréter notre calcul. 

Essayons maintenant de voir exactement quel chemin est le bon pour la phase de reconnaissance. Essayons le payload suivant : 

`{{7*7}}`
![](../../img/Pasted%20image%2020260516172945.png)

On peut voir que celui-ci n'as pas été interpréter, donc on ne prendras pas le chemin du bas mais celui du haut .

Testons maintenant le payload suivant : 

`a{*comment*}b`


![](../../img/Pasted%20image%2020260516173306.png)

Celui-ci n'as pas été interpréter. Donc nous allons passer à la suivante : 

`${"z".join("ab")}`

![](../../img/Pasted%20image%2020260516173431.png)

Nous pouvons voir que celui-ci à bel est bien été interpréter par le serveur ce qui veut dire qu'on est sur du Mako.

Bravo vous venez trouvez de vous même comment savoir quel est le vecteur d'attaque que nous allons utiliser pour cette SSTI.

Maintenant passons à la phase d'exploitation.

![](../../img/Pasted%20image%2020260516173557.png)

Maintenant qu'on sais qu'on est sur du mako et que mako est basé sous python allons dans la rubrique Python.md. 

![](../../img/Pasted%20image%2020260516173710.png)


Maintenant qu'on est sur Mako et qu'on veux effectuer une RCE nous allons tentez les différentes solution qui nous sont présenté. (Pas toutes).

![](../../img/Pasted%20image%2020260516173803.png)

Je vais tenter de copier coller pour la première, la deuxième mais aussi pour la 10ème voir ce que ça donne.

Malheureusement aucune nous renvoie ce qu'on voudrais. C'est pour cela que je vais utiliser l'option de l'obfuscation (à priorisé en général pour faire un contournement du WAF)
Le WAF(Web Application Firewall) va examiné notre requête et la jeté ou soit la gardé. C'est pour cela que je vais décidé de tenter de le bypass via l'obfuscation. 

Pour cela utilisons le payload ci dessous.

```python
${self.module.cache.util.os.popen(str().join(chr(i)for(i)in[105,100])).read()}
```

ici cela va utiliser os.popen qui permet d'exécuté une commande via la librairie os, par la suite on dis qu'on va prendre tout ce qui se trouve à l'intérieur et que ce sera sous la forme d'un str. le .join permet de joindre les différents caractère qui se trouve dans la boucle. Ici chr transforme toute les valeurs de i en caractère en traduisant via le tableau ascii. I prend ici les valeurs 105 (i) et 100 (d) et .read() permet de renvoyer lire le résultat de la commande. 

Globalement cela va exécuté la commande "id" dans le input et nous renvoyer le résultat de la commande. 

testons maintenant : 

![](../../img/Pasted%20image%2020260516174845.png)

Bingo!

Maintenant il faut qu'on exécute et qu'on regarde les fichiers ce trouvant dans le répertoire racine. 


Pour cela on va faire ls -la / 

Ce qui donne dans le payload : 

```python
${self.module.cache.util.os.popen(str().join(chr(i)for(i)in[108,115,32,45,108,97,32,47])).read()}
```


![](../../img/Pasted%20image%2020260516175508.png)

Bingo Bon c'est pas trop joli regardons dans un fichier texte de base. 

![](../../img/Pasted%20image%2020260516175545.png)


C'est plus jolie mais on peut voir surtout que sur la troisième lignes nous avons flag.txt qui se trouve dans le répertoire racine comme dans le code source. On va pouvoir maintenant le lire en faisant : 

cat /flag.txt

Pour tout ce qui est traduction de text en ascii j'utilise ce site : 

https://www.browserling.com/tools/text-to-ascii

99 97 116 32 47 102 108 97 103 46 116 120 116

Voici la chaine renvoyer lorsqu'on fait cat /flag.txt sur le site. 


Maintenant il suffit de le mettre dans notre payload. 

```python
${self.module.cache.util.os.popen(str().join(chr(i)for(i)in[99,97,116,32,47,102,108,97,103,46,116,120,116])).read()}
```

![](../../img/Pasted%20image%2020260516180044.png)

Bingo ! 

Prenons nos points. 

![](../../img/Pasted%20image%2020260516180102.png)

