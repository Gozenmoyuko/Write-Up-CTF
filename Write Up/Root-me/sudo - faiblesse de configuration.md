Étape 1: 
![Pasted image 20250329171927.png](../img/Pasted%20image%2020250329171927.png)
Ici j'utilise **whoami** pour savoir qu'elle utilisateur nous somme actuellement sur le SSH.

Étape 2: 

![/img/Pasted image 20250329172008.png](../img/Pasted%20image%2020250329172008.png)
Puis j'utilise **ls -la** pour pouvoir réperer tout les fichiers + fichiers cacher qui sont actuellement sur le réseau SSH.
___
Étape 3:
![Pasted image 20250329172104.png](../img/Pasted%20image%2020250329172104.png)
	J'exécute le readme.md pour voir sont contenu et voir si cela pourra nous aider pour la suite
___

Étape 4:
![Pasted image 20250329172352.png](../img/Pasted%20image%2020250329172352.png)
	Ici **sudo -l** va nous permettre de listez les commandes qui sont possible à utiliser et les utilisateur et leurs commandes qui peuvent utilisées .
	Ici on vois que l'utilisateur **app-script-ch1-cracked** peut utiliser que ces commandes à la suite **/bin/cat /challenge/app-script/ch1/notes/*** c'est pour cela que nous feront un retour par la suite pour accéder au fichier qui nous est demander 
___

Étape 5:
![Pasted image 20250329172453.png](../img/Pasted%20image%2020250329172453.png)
	**sudo -u** pour préciser l'utilisateur suivie du binaire de cat en indiquant le chemin absolue et on met la commande qui nous à été donné dans le sudo -l. ici **/bin/cat /challenge/app-script/ch1/notes/*** puis le **/../** nous fais un retour ici ils nous ramène à **ch1** et nous montres ce que contient ce dossier. 

___

Étape 6:
![Pasted image 20250329172530.png](../img/Pasted%20image%2020250329172530.png)
	Ici on reprend notre commande **sudo -u** car on va utilisé l'utilisateur qui à les droits d'utiliser **/bin/cat /challenge/app-script/ch1/notes** qui est **app-script-ch1-cracked** puis ont retourne dans le fichier ch1 pour faire notre manipulation car on été obligé de marquer ce qui nous été indiquer dans **sudo -l** et donc on retourne pour pouvoir après aller la où se trouve le flag qu'on a remarquer en lisant le readme.md et on l'ouvre en faisans **/.passwd** 