
Dans ce repo vous verrez comment utiliser la librairie scapy pour pouvoir vous même manipuler des trames de types Ethernet, Wifi etc...

Scapy est une librairie très puissante qui permet de manipuler vous mêmes les trames, vous pouvez donc faire plusieurs attaques de vous mêmes en vous documentant directement sur leur documentation qui est ci-dessous : 

https://scapy.readthedocs.io/en/latest

Maintenant que vous avez la documentation je vais vous montrez comment simplement vous pouvez vous y retrouvez mais aussi comment faire en sorte de pouvoir facilement créer des scripts que nous souhaitons via Scapy.


## Installation de Scapy

*NB: Pour l'installation je le fais directement sur mon wsl, il y aura donc une spécificité sur le install*  


Pour l'installation il vous suffit de faire : 
```
pip install scapy
```

Pour ma part je fais via --break-system-packages 

![](../Write%20Up/img/Pasted%20image%2020260915125059.png)

Pour les codes d'erreur ne vous en faites pas c'est parce que je suis wsl.

si vous souhaitez l'installer en devellopement version je vous invite à lire la documentation très clair  : 
https://scapy.readthedocs.io/en/latest/installation.html

## Lancement de scapy

Pour lancer scapy rien de plus simple, vous allez taper sudo scapy

Pour ma part je suis sur wsl donc je fais : 
```
sudo /usr/local/bin/scapy
```


![](../Write%20Up/img/Pasted%20image%2020260915125351.png)

Maintenant que vous avez fais cela (lancer scapy) passons à son utilisation.


## Utilisation de scapy

Maintenant nous allons voir comment utiliser scapy et notamment comment s'y retrouver pour nos différents type de script (on verra le cheminement de penser)


Donc tous d'abord il faut avoir une idée protocolaire de ce que vous voulez faire, par exemple moi je veux faire une attaque de deauth, dès que vous savez ce que vous voulez faire, il faut comprendre sur quel couche du model OSI vous voulez fabriquer votre paquet/trame. Bien-sûr il vous ai possible de construire un paquet avec la couche 2 puis 3 etc... Il suffit de séparer via "/" lors de votre création de votre paquet dans une expression python.

Exemple : 

```python
paquet = (
	Couche2()/
	Couche3()/ #Ou suite de la trame de couche deux vous pouvez faire les entêtes etc...
	Couche4()/
	...
	...
	Couche7()/
)
```












## Tutoriel option























## Projet



`sudo apt update && sudo apt install aircrack-ng -y`
