Nous somme sur un challenge plutôt facile de pico-ctf qui va consister à jouez avec les différentes requête pour pouvoir accès au compte voulut. 


Dans l'énoncer il nous ai donner un e-mail de compte mais malheureusement pas le mot de passe.

![../img/Pasted image 20260307121916.png](../img/Pasted%20image%2020260307121916.png)


Il va falloir donc avoir accès à ce compte pour pouvoir trouver le flag.


Pour cela ouvrez le site qui vient d'être générer lorsque vous avez cliquez sur "Start Instance" en dessous il y a un lien "here".


![../img/Pasted image 20260307122028.png](../img/Pasted%20image%2020260307122028.png)

Voici comment ce présente le site Web qui à été générer.

Je met donc l'email et je tente un mot de passe dont j'ai mis "test".

Bon malheureusement ce n'est pas le mot de passe du compte. 

Lorsque l'on clique login rien ne se passe.

Allons analyser le code source de la page pour voir s'il n'y as pas quelques choses d'intéressant car oui si vous regardez bien vous pouvez voir que dans l'énoncer il y est écrit que le développeur à laisser quelque chose de secret sur le site. 

![../img/Pasted image 20260307122538.png](../img/Pasted%20image%2020260307122538.png)

Mhm intéressant, le développeur ou ceux qui devais mettre le site en production on oublier d'enlever le commentaire juste au dessus.

Cela ressemble beaucoup à du code cypher. Copier le texte et mettez le sur ce lien : 

https://www.dcode.fr/caesar-cipher


![../img/Pasted image 20260307130449.png](../img/Pasted%20image%2020260307130449.png)

Et voici ce qui est écrit après le brute force du cypher. 

Bon que nous savons qu'il faut utiliser un paramètre dans le Header spécifique allons sur BurpSuite et récupérer le lien lorsque vous appuyez sur login sur le site. 

Pour ce faire allez sur Proxy > Intercept > Open browser



![../img/Pasted image 20260307130635.png](../img/Pasted%20image%2020260307130635.png)



![../img/Pasted image 20260307130719.png](../img/Pasted%20image%2020260307130719.png)

Un navigateur Chromium à été lancer maintenant mettez le lien du site. 

![../img/Pasted image 20260307130823.png](../img/Pasted%20image%2020260307130823.png)


Remettez l'email et le mot de passe AVANT DE CLIQUEZ SUR LOGIN RETOURNEZ SUR VOTRE BURPSUITE ET METTEZ BIEN "INTERCEPT ON"  en cliquant sur "intercept off" !!. 

Maintenant cliquez sur login. 

![../img/Pasted image 20260307130935.png](../img/Pasted%20image%2020260307130935.png)

Vous venez de récupérer la requête entre le serveur et le client. Si c'est votre première fois avec Burpsuite je vais vous expliquez à quoi ça sert ce qu'on vient de faire.

Burpsuite va devenir un proxy (Il va se placer entre vous et le serveur) pour récupérer votre requête (l'action que vous venez de faire exemple : POST, GET, PUT, OPTIONS etc... ) et vous le recevez sur votre onglet d'interception.

Schéma explicatif : 




Ce que vous pouvez faire c'est Clique Droit > Send to Repeater

Cela va nous permettre de modifier notre requête manuellement. 

Comme rajouter l'Header qui nous est donné dans la note : 

![../img/Pasted image 20260307131332.png](../img/Pasted%20image%2020260307131332.png)


Bon voici ce qui nous à été récupérer, c'est une situation plutôt lambda on à une requête POST (on va déposer quelques choses et pas GET qui lui récupère quelque chose comme une page WEB par exemple)

en paramètre de la requête on à l'email et le mot de passe qui à été utiliser. 

À votre droite vous pouvez voir la réponse, ici nous voyons que notre requête n'as pas pu ce faire : "success": false
mais aussi que nous avons une erreur qui veut dire que nous avons pas l'autorisation d'accéder au compte. C'est car nous avons pas le bon mot de passe. Mais vous enfaite pas on va le récupérer c'est justement ça le but du CTF le flag est le password.

Bref continuons vous allez voir c'est simple vous allez juste reprendre l'header qui nous à été donner par le développeur et le mettre après le Content-Type de préférence. 

Pourquoi on utilise le Header ? car pour l'instant c'est la seule solutions que nous avons qui est la plus simple avant de parler de XSS possible ou SQL ou autre type d'injection de commande. 

Mais aussi car si vous comprenez un minimum l'anglais vous auriez du comprendre lorsque vous avez lu le code cypher decoder que ce header permet au développeur de bypass pour avoir un accès en cas de problème.  

Bon passons au chose sérieuse. 

![../img/Pasted image 20260307132025.png](../img/Pasted%20image%2020260307132025.png)

Voici à quoi doit ressembler votre nouvelle requête regarder la ligne 7 de notre requête. 

Voyons si cela à marcher. 

![../img/Pasted image 20260307132125.png](../img/Pasted%20image%2020260307132125.png)

Bingo vous venez de faire votre premier Bypass d'authentification mais aussi de réussir le challenge. J'ai cacher exprès ne m'en voulais pas mais il faut que vous apprenez et pas juste copier coller le flag. On se revoit pour d'autre challenge de Pico-CTF plus dur !! 
