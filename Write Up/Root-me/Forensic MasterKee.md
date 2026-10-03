
Aujourd'hui je m'attaque à un nouveau challenge de forensic. 

Je rappelle que l'analyse forensique n'est pas forcément ma spécialité. À la base je suis dans tout ce qui est sécurité WEB serveur et client. Mais il y a peu, je me suis intéressé au reverse engineering, à l'assembleur, etc...

Et je me suis dit "Pourquoi pas s'attaquer au Forensique

Aujourd'hui je suis content de vous montrer le challenge MasterKee, car j'ai pu comprendre comment la CVE derrière tout cela marche et je vais vous l'expliquer.


Tout d'abord, attaquons-nous à l'énoncer.


![](../img/Pasted%20image%2020261003213102.png)

Nous pouvons actuellement voir que notre collègue ce croit INTOUCHABLE.

Nous allons lui montrer que ce n'est pas parce qu'on est sur Keepass que tout est forcément sécurisé ;) 


Nous allons actuellement installer le fichier : 

![](../img/Pasted%20image%2020261003213638.png)


Le ficher est en ZIP et donc j'ai extrait le fichier.

Nous pouvons voir que nous avons un fichier .DMP qui va nous permettre de lui montrer qu'il n'est pas invincible. 


Bon commençons. 

![](../img/Pasted%20image%2020261003214127.png)

On peut remarquer que le fichier date de 2024, nous allons voir si une CVE se trouve dans les plages de versions ou de date de ce fichier !

Bon, on sait que notre but est d'extraire un mot de passe pour le keepass pour pouvoir récupérer le flag du challenge (facile à deviner car on a aussi le keepass dans le zip).

Bon, cherchons sur internet la CVE. 

![](../img/Pasted%20image%2020261003223751.png)

Ici, nous pouvons voir qu'il y a une CVE de 2023, mais ce n'est pas parce que la CVE date de 2023 que forcément il n'est pas compromis par cette CVE. 

Car oui, plusieurs personnes ne mettent pas forcément leur système à jour ou autre.

Donc, nous allons partir sur cette CVE et si ce n'est pas ça alors ce n'est pas grave, nous saurons que lorsqu'on tombe dans le même cas que ce n'est pas cette CVE-ci.


![](../img/Pasted%20image%2020261003224248.png)

Je vous fais gagner du temps, en dessous nous pouvons voir un lien GitHub, qui est directement la CVE que nous cherchons et si vous vous rendez sur le site de NIST (le premier lien), vous verrez qu'il nous renvoie sur le GitHub.


![](../img/Pasted%20image%2020261003224629.png)

https://github.com/vdohney/keepass-password-dumper

Voici le lien pour l'exploit. 

Pour l'installation, il vous suffit de suivre les étapes d'installation qui sont écrites sur le GitHub directement.

Bon maintenant passons à l'exploitation. 

Si vous avez bien fait l'installation, vous devriez normalement installer Dotnet. 

C'est ce qui va nous permettre de lancer notre script. 

Rendez-vous dans le dossier de la CVE :

![](../img/Pasted%20image%2020261003225528.png)

Puisque je suis sous WSL, j'ai dû installer différemment de vous, et donc je dois lancer depuis le fichier ~/.dotnet/dotnet mais dans votre cas il vous suffit de le lancer tel que : 

```bash
dotnet run ../MasterKee.DMP
```

Il est important de se situer dans le fichier du repo de la CVE que nous avons installé. 

Car sinon le lancement Dotnet ne marchera pas.

Bon, voyons ce que la CVE nous a trouvé comme clé maître : 


![](../img/Pasted%20image%2020261003225917.png)

Je cache le début pour éviter la tricherie, mais croyez-moi sur parole lorsque je vous dis que le début de la clé maître n'est pas certain. On peut le voir notamment avec le {e, 3, .....} qui montre que la CVE n'a pas trouvé. 

Personnellement, au vu des lettres qui sont proposées (qui ont été retrouvées dans la RAM), je sais que c'est la lettre H car c'est celle qui fait sens avec le mot de passe.

Bon maintenant je vais le rentrer dans le fichier keepass : 

![](../img/Pasted%20image%2020261003230343.png)


Bingo, je suis à l'intérieur, maintenant il suffit de cliquer sur "Copy password" et de le mettre sur root-me : 

![](../img/Pasted%20image%2020261003230435.png)


![](../img/Pasted%20image%2020261003230552.png)

Bingo, on a le bon format de flag, allons voir maintenant si c'est bon : 

![](../img/Pasted%20image%2020261003230728.png)



Maintenant, rappelez-vous que je vous avais dit comme quoi nous pouvions facilement recréer cette CVE lorsqu'on a compris comment marche la version antérieure à la 2.54 de keepass.

Je vais vous expliquez.

La CVE repose sur un défaut visuel de KeePass. Lorsque l'utilisateur tape son mot de passe au clavier, le logiciel crée involontairement une copie en clair de ce qui est écrit dans la mémoire RAM à chaque fois qu'une nouvelle lettre est ajoutée.

Il suffisait donc de faire un dump de la mémoire pour retrouver ces morceaux de texte enregistrés au fur et à mesure de la frappe.


Ceci-dit, il restait le premier caractère, comme nous avons pu le voir qui est caché par le caractère  ●
En effet, l'ancien caractère est caché, mais il suffit de toutes les coordonnées pour retrouver (ce que fait actuellement la CVE si vous regardez bien le code source) . 

Mais il reste tout de même une solution plus simple, il vous suffit de chercher grâce à la commande "strings" les chaînes de caractères stockées dans le dump de la mémoire et simplement de retrouver les chaînes. 

Rappelez-vous petite subtilité, les systèmes lisent en Little Endian (Les bits de poids faible à gauche et les bits de poids fort à droite) c'est pour cela que nous allons utiliser la commande : 

```bash
strings -e l ../MasterKee.DMP
```

Ici -e permet de spécifier au programme l'encodage qu'il doit chercher et le l  est utilisé pour le Little Endian par strings.

Bon, en réalité c'est assez complexe de mettre en place directement la lecture du fichier, dans la cve il fait une recherche du caractère ● qui se traduit en \xCF\x25 et c'est ainsi qu'il retrouve les caractères suivants.

Mais pour trouver le caractère premier il vous suffit par la suite de faire un grep avec ce que vous connaissais : 


![](../img/Pasted%20image%2020261003233606.png)

Le premier caractère est montré car le dump mémoire directement est plus puissant pour chercher le caractère que le script dotent car il fait des estimations, or ici vous pouvez le voir directement. Vous combinez les deux types et vous êtes gagnant à 100%, et vous ne faites pas de la devinette. La mémoire ici est plus puissante car nous avons exploité une autre erreur qui a été le copier-coller du mot de passe et donc il se situe dans le : 
CClipDataObject::GetDataHereImpl






