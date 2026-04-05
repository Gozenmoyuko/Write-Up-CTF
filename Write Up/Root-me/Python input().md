

Tout d'abord connecter vous sur le challenge :
ssh -p 2222 app-script-ch6@challenge02.root-me.org

![[Pasted image 20260308115226.png]]

Mot de passe du ssh : app-script-ch6

Par la suite il faut connaître l'environnement où on se situe, qu'elle droit on as etc... C'est pour cela que nous allons effectuer ls -la puis whoami et enfin pwd pour savoir où nous nous situons

![[Pasted image 20260308115247.png]]

![[Pasted image 20260308115302.png]]

![[Pasted image 20260308115322.png]]

Bon maintenant que nous avons toute c'est informations voyons ce qu'on peut faire. On voit que sur le ch6.py nous pouvons juste lire le script python et l'exécuté. 

Par la suite on voit qu'il y a un test d'identifiant grâce au setuid-wrapper qui est une fonction linux. On à le script en .c que l'ont peut lire et ça version compiler qu'on peut exécuté. 

Et enfin on voit qu'il y a un fichier qui s'appel .passwd que nous pouvons rien faire mais que le system lui peut lire. 

On va donc regarder le script python.

![[Pasted image 20260308115420.png]]

On peut voir que la première fonction ne sert qu'a afficher un message d'erreur du défi.

Par la suite il demande à l'utilisateur de rentrer un mot de passe qu'il stock dans p et si c'est faux alors il renvoie la fonction youLose()

et enfin il ouvre le fichier .passwd le stocke dans f et enlève les \n et stocke le mot de passe dans passwd.

Si le password est le même que celui rentrer par l'utilisateur alors on peu valider le CTF grâce au même rentrer.

Mais comment faire pour exploiter cela ?

Enfaite le input n'est pas filtrer il demande juste à l'utilisateur de rentrer un password donc un str ou un int ou float et donc l'utilisateur peut rentrer un nom de variable déjà existante et lire une variable ou autre.

Pour ce challenge nous allons faire simple nous allons voir si un autre fichier n'appelles tout simplement pas notre fichier ch6.py.

Pour cela lisons le fichier en .c qui est setuid-wrapper.c .

![[Pasted image 20260308115640.png]]

Pas besoin de tout comprendre en gros il exécute directement le script ch6.py et on peut essayer de lancer ça version compiler avec gcc ou autre (setuid-wrapper) pour voir.

![[Pasted image 20260308115800.png]]

Okay parfait tentons d'injecter du code directement ici.

![[Pasted image 20260308115826.png]]

Okay on vois que le script nous renvoie bien ce que j'ai fais et qu'il quitte le script c'est donc qu'il s'exécute et affiche notre message au lieu du "Try again". 

Mais pourquoi `__import__` en python la fonction `__import__` permet d'appeler un module ou "package" lorsque nous appelons une variable en gros si je fait :

`nom_du_module = "os"` 
`os = __import__(nom_du_module)`

autrement dis c'est comme faire un simple `import os` sur python.

Alors ici nous aurons le module "os" qui sera importé dans notre script et stocker dans ma variable "os". 

Maintenant .system qu'est-ce que c'est ? 

C'est une fonction du module "os" c'est pour cela qu'on l'enchaîne directement après le `__import__("os")` En réalité c'est donc comme si je faisais os.system(....) 

Exemple du fonctionnement de os.system : 

`import os os.system("mkdir test")` 

Ici cela va créé un fichier mkdir car le module "os" utilise du langage de bas niveau comme du bash et donc si je fais `cat` ça lira un fichier si je fais echo répetera ce que j'ai marquer etc.... C'est pour cela que lorsque j'ai taper :

`./setuid-wrapper` 
`Please enter password :` 
`__import__("os").system("echo testtt")` 
`testtt`


Ici j'ai donc exécuté le fichier .c compiler j'ai mis le payload que j'ai expliquer donc importé os puis utiliser os.system pour faire des commandes en bash et donc echo répète testtt.

Maintenant que vous avez compris essayons de voir si on peut tout simplement faire cela avec cat pour ouvrir le fichier qui contient notre flag qui est .passwd comme on peut le voir dans le script python

![[Pasted image 20260308120511.png]]

Bingo ici on voit bel est bien que le fichier à été exécuté par le system et que celui-ci c'est donc exécuté et que nous avons le flag.

J'ai masquer à vous de faire, essayer sans regarder !!!
