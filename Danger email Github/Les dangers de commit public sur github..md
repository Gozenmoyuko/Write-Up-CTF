
Il y a quelques choses que plein d'utilisateurs sur GitHub ne savent pas et qui ne se soucient pas de savoir. Puis lorsqu'ils sont cibles de phishing ou même cibles d'osant ciblé ou de C-sint ciblé, tel que l'exploitation de base de données leak ou acheter sur des forums de cybercriminels. 

Dans le fonctionnement de GitHub Basic, lorsque vous n'essayez pas de comprendre vous-même ou lorsque vous n'êtes pas dans le domaine de l'osint ou même êtes victime de osint vous pouvez facilement vous demander : "Comment une personne à qui j'ai seulement donné mon GitHub a réussi à avoir mon email... 

Il faut savoir que votre email est partagé directement lorsque vous faites un commit public via Git, par exemple. Mais aussi, il faut savoir que le paramètre de protection du partage de votre email par CLI est à la base désactivé et je vous propose de l'activer, mais encore mieux de changer les settings Git pour ne plus partager votre email personnel ! 


## Étape 1 : Retirer l'envoi d'email par push et garder son adresse mail privée.

Pour cela nous allons faire simple et efficace.

Tout d'abord rendez-vous sur https://github.com/settings/emails , si l'endpoint à changer alors cliquez sur Settings > Emails.

![](../Write%20Up/img/Pasted%20image%2020260716081400.png)

Je vais parler avec les chiffres que j'ai indiqué et vous expliquez ce qu'il faut faire.

- Tout d'abord, regardez que le bouton 1 est bien activé. 
- 2 Nous allons activer ce paramètre qui est très important, mais cela ne suffit pas. Vous savez très bien qu'il ne faut pas faire confiance au bouton, c'est comme la télémétrie sur Windows, vous avez beau la désactiver, elle sera toujours là !! 
- 3, Ce sera l'email que nous allons configurer sur notre GitHub pour les commits. C'est un email rattaché à GitHub qui n'a aucun sens technique car vous ne pourrez pas l'utiliser pour l'enregistrement de compte Google ou autre par exemple, donc aucune email reliée à vos comptes personnels ! 

Résultat, il faut que vos paramètres ressemblent à cela : 

![](../Write%20Up/img/Pasted%20image%2020260716081817.png)


Maintenant vous pouvez voir que nous l'avons activé. 


## Étape 2 : Configuration du git local.

Tout d'abord, nous allons voir quel email est configuré dans notre configuration de Git local.

Pour cela nous allons taper la commande suivante : 
```bash
git config user.email
```

![](../Write%20Up/img/Pasted%20image%2020260716082029.png)
Comme vous pouvez le voir, j'ai flouté mes informations personnelles. Mais vous pouvez voir qu'il y a bel et bien mon adresse e-mail privée.


Maintenant nous allons configurer avec le mail que nous avons dans les paramètres d'avant pour cela on fait : 
```bash
git config --global user.email "90153990+Gozenmoyuko@users.noreply.github.com"
```
![](../Write%20Up/img/Pasted%20image%2020260716140537.png)

Maintenant nous allons vérifier comme quoi les settings ce sont bien appliquer. 
![](../Write%20Up/img/Pasted%20image%2020260716141042.png)
