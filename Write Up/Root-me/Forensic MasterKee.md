
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

