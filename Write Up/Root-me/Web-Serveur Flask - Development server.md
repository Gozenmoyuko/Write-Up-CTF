
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

![](../img/Pasted%20image%2020260425163715.png)

On peut voir que lorsqu'on trie en faisant CTRL + F et qu'on cherche "web-app" on trouve beaucoup de correspondance avec /home/web-app/.local/lib/python3.11/site-packages/

Avant de récupérer le chemin complet du processus qui est exécuté on va aller regarder qu'elle fichier à eu une commande d'exécution pour cela on va visité le fichier suivant(/proc/self/cmdline ) : 
![](../img/Pasted%20image%2020260425170259.png)
Okay le processus s'appel app.py on aura donc quelques chose de la forme : 
/home/web-app/.local/lib/python3.11/site-packages/[dependance]/app.py
dans ce que j'appel dépendance je parle notamment de ce qui est dans un requirements.txt basic. 

Pour cela il faut savoir que le requirements.txt ce trouve forcément quelques part dans le répertoire du processus actif. Comme je vous l'ai dis /proc est le dossier des interfaces virtuelle vers les informations du processus. /self est le processus actif donc ici app.py et cwd (**C**urrent **W**orking **D**irectory) est le répertoire dans lequel notre processus est entrain de tourner actuellement, c'est un symlinks (lien symbolique) important puisqu'il nous enverra toujours dans le dossier du processus actuelle puis que le principe d'un lien symbolique je vous le rappel est de renvoyer vers un autre dossier ou fichier.

Ici /proc/self/cwd va forcément nous renvoyer sur le répertoire du processus il suffira de taper /requirements.txt pour voir les dépendances de l'application. 

![](../img/Pasted%20image%2020260425171914.png)

Okay parfait on voit dans les dépendances donc qu'il y a notre Werkzeug qu'on avait trouver, mais aussi maintenant on voit quelques choses qui nous ai familier qui est Flask. 

Essayons de voir ce que nous renvoie si on lit le fichier donc qui je rappel été /home/web-app/.local/lib/python3.11/site-packages/[dependance]/app.py

ici on va remplacer dependance par flask. 

![](../img/Pasted%20image%2020260425172148.png)


Okay on voit que c'est bien flask qui est utiliser. Mais avant tout, il y a beaucoup trop d'information nan ? Essayons de rejouer avec notre symlink(liens symboliques) pour voir ce que contient notre fichier app.py directement. Car la c'est plein de choses qui sont regrouper mais pas forcément le plus important. 

On va donc taper : 

/proc/self/cwd/app.py 

![](../img/Pasted%20image%2020260425172323.png)
Okay Bingo ! on voit tout en bas qu'une option de debug qui est utiliser(True) sur notre serveur web donc il y a une shell de debug du serveur web. Mais comment connaitre le endpoint de ce shell de Werkzeug ? Tout simplement on appel notre ami internet :)

![](../img/Pasted%20image%2020260425172522.png)

Bingo cherchons la de dans. 

![](../img/Pasted%20image%2020260425172543.png)

Tiens bingo il y a un danger qui est marquer. Il ne faut pas activer le debug sur des machines en productions... Pas pro tout cela.

Bon continuons essayons de voir ou ce situe cette console de debug : 

![](../img/Pasted%20image%2020260425172910.png)

Essayons de voir s'il est bien actif sur notre site pour cela je vais taper l'url suivante : 

challenge01.root-me.org:59085/console
