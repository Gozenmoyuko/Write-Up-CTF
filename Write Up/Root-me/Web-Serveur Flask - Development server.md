
Pour ce challenge il faudra récupérer le flag qui se situe dans le répertoire web de l'application :

![](../img/Pasted%20image%2020260425155602.png)

Voyons voir à quoi ressemble le site web : 

![](../img/Pasted%20image%2020260425155628.png)


Cliquons sur "Nos services" (Our services) : 

![](../img/Pasted%20image%2020260425155851.png)

On arrive sur une page qui n'est pas totalement fonctionnel soit disant leurs services  de recherches de d'autres services n'est pas totalement fonctionnel. Essayons de chercher quelques chose comme "test". 

![](../img/Pasted%20image%2020260425155957.png)

Bizarre, il essaye de chercher un fichier ou de l'ouvrir, essayons de rentrer dans le répertoire racine de la machine : 

![](../img/Pasted%20image%2020260425160055.png)

Okay donc il ne rentre pas dans un fichier peut-être qu'il exécute un "cat" ou autre, pour en avoir le coeur net essayons lire le fichier /etc/passwd en tapant dans l'input /etc/passwd : 

![](../img/Pasted%20image%2020260425160219.png)

Bingo ! On voit à la toute fin qu'il y a un utilisateur linux qui s'appel web-app car il détient un dossier utilisateur (/home) donc la suite est le nom de l'utilisateur, ici qui est web-app. On à déjà un utilisateur :)

Maintenant essayons de voir qu'elle interface réseaux dispose ce serveur, vos mieux prendre un max d'information sur le serveur pour cela on va lire le fichier /proc/net/arp qui contient tout les processus arp de cette machine : 

![](../img/Pasted%20image%2020260425160550.png)

Ici on voit que l'interfaces réseau utiliser de la machine est le "eth0"

Essayons de récupérer l'adresse mac dans le fichier /sys/class/net/eth0/address : 

![](../img/Pasted%20image%2020260425161404.png)

Okay parfait, maintenant essayons de voir qu'elle environnement web est utiliser. Pour cela on va visiter le fichier /proc/self/environ(au passage /proc est souvent le dossier des interfaces virtuelle vers les informations du noyau des processus. Le /self est le processus actuel (celui qui lit le fichier et le /environ est désigner pour les variables d'environnement )) : 

![](../img/Pasted%20image%2020260425161459.png)

On voit que le serveur utilise WERKZEUG essayons d'aller voir leurs documentation : 

https://werkzeug.palletsprojects.com/en/stable/tutorial/

Voici l'utilisation de base de werkzeug et comment il marche. Mais pour l'instant voyons voir d'autre fichier qui peuvent être intéressant.

Voyons voir le stockage des adresses mémoire des différents processus dans /proc/self/maps : 

![](../../Pasted%20image%2020260425163715.png)

On peut voir que 