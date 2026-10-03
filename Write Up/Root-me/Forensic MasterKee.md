
Aujourd'hui je m'attaque à un nouveau challenge de forensic. 

Je rappelle que l'analyse forensique n'est pas forcément ma spécialité. À la base je suis dans tout ce qui est sécurité WEB serveur et client. Mais il y a peu, je me suis intéressé au reverse engineering, à l'assembleur, etc...

Et je me suis dit "Pourquoi pas s'attaquer au Forensique

Aujourd'hui je suis content de vous montrer le challenge MasterKee, car j'ai pu comprendre comment la CVE derrière tout cela marche et je vais vous l'expliquer.


Tout d'abord, attaquons-nous à l'énoncer.


![](../img/Pasted%20image%2020261003213102.png)

Nous pouvons actuellement voir que notre collègue ce croit INTOUCHABLE.

Nous allons lui montrer que ce n'est pas parce qu'on est sur Keepass que tout est forcément sécurisé ;) 


Nous allons actuellement installer le fichier : 

![](../../Pasted%20image%2020261003213638.png)


Le ficher est en ZIP et donc j'ai extrait le fichier.

Nous pouvons voir que nous avons un fichier .DMP qui va nous permettre de lui montrer qu'il est pas invisible 