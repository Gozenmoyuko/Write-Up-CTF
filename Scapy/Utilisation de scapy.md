
Dans ce repo, vous verrez comment utiliser la librairie scapy pour pouvoir vous-même manipuler des trames de types Ethernet, Wifi, etc...

Scapy est une librairie très puissante qui permet de manipuler vous-mêmes les trames, vous pouvez donc faire plusieurs attaques de vous-mêmes en vous documentant directement sur leur documentation qui est ci-dessous : 

https://scapy.readthedocs.io/en/latest

Maintenant que vous avez la documentation, je vais vous montrer comment simplement vous pouvez vous y retrouver, mais aussi comment faire en sorte de pouvoir facilement créer des scripts que nous souhaitons via Scapy.


## Installation de Scapy

*NB : Pour l'installation, je le fais directement sur mon wsl, il y aura donc une spécificité sur l'installation*  


Pour l'installation il vous suffit de faire : 
```
pip install scapy
```

Pour ma part je fais via --break-system-packages 

![](../Write%20Up/img/Pasted%20image%2020260915125059.png)

Pour les codes d'erreur, ne vous en faites pas, c'est parce que je suis wsl.

si vous souhaitez l'installer en "developpement version", je vous invite à lire la documentation très claire : 
https://scapy.readthedocs.io/en/latest/installation.html

## Lancement de scapy

Pour lancer scapy rien de plus simple, vous allez taper sudo scapy

Pour ma part, je suis sur wsl donc je fais : 
```
sudo /usr/local/bin/scapy
```


![](../Write%20Up/img/Pasted%20image%2020260915125351.png)

Maintenant que vous avez fais cela (lancer scapy) passons à son utilisation.


## Utilisation de scapy

Maintenant nous allons voir comment utiliser scapy et notamment comment s'y retrouver pour nos différents types de script (on verra le cheminement de penser)


Donc tout d'abord, il faut avoir une idée protocolaire de ce que vous voulez faire, par exemple moi je veux faire une attaque de début, dès que vous savez ce que vous voulez faire, il faut comprendre sur quelle couche du modèle OSI vous voulez fabriquer votre paquet/trame. Bien sûr, il vous est possible de construire un paquet avec la couche 2, puis 3, etc... Il suffit de séparer via "/" lors de votre création de votre paquet dans une expression Python.

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

Maintenant que vous savez tout cela, nous allons voir un tutoriel rapide sur les options qui peuvent être proposées par Scapy et leurs utilités.

Comme vous l'aurez compris, pour lancer l'interpréteur scapy vous devez effectué la commande : 
```bash
sudo scapy
```

Par la suite, nous allons maintenant voir les différentes commandes dans l'interpréteur qui vont beaucoup nous aider.

les commandes de type listing : 

- **ls()** : Permet d'afficher tous les protocoles que scapy connaît avec une courte description.

- **ls(Protocole)** : Permet d'afficher chaque champ du protocole choisi. **Réflexe à avoir systématiquement avant de construire un paquet d'un protocole dont vous n'avez jamais maîtrisé ou dont vous avez du mal à maîtriser :  

- **lsc()** : Permet d'afficher toutes les fonctions utilitaires de haut niveau fournies par Scapy telles que arping, wrpcap, etc... 

- **explore()** : Ouvre une interface interactive dans la console qui permet de naviguer dans l'arborescence des protocoles et des modules disponibles. Attention pour l'utilisation de explore(), il faut installer iPython sinon vous allez tomber sur : 
"ImportError: prompt_toolkit is not installed ! You may install IPython, which contains it, via `pip install ipython`"

- **help()** : Permet sur n'importe quel objet d'afficher la docstring, généralement plus complète que le système de listing. 

Si IPython est installé, vous pouvez utiliser aussi l'option 
"?" et "??".

- IP? : Permet d'afficher la docstring rapide
- IP?? : Permet d'afficher le code source complet de la méthode ou de la fonction.


Exemple :
- **ls(DNS)** ce qui donne : 

![](../Write%20Up/img/Pasted%20image%2020260927191751.png)


**lsc()** ce qui donne : 
![](../Write%20Up/img/Pasted%20image%2020260927192030.png)

**explore()** Ce qui donne : 
![](../Write%20Up/img/Pasted%20image%2020260927194244.png)

- help() ici je vais vous montrez avec IP, help(IP) ce qui donne : 
-![](../Write%20Up/img/Pasted%20image%2020260927194627.png)

## Projet



```bash
sudo apt update && sudo apt install aircrack-ng -y
```

```bash
sudo airmon-ng
```

