
![](../Write%20Up/img/MomuScan_final.jpeg)



Tout d'abord, nous allons initialiser notre environnement, pour cela nous allons utiliser Docker qui va nous permettre, via une image, de créer un conteneur. Grâce à ce conteneur, nous allons pouvoir créer d'office plusieurs vulnérabilités et tenter de les scanner sans devoir être dans l'illégal en l'essayant sur des sites publics. Ici le projet restera strictement local ou sur des serveurs dont nous avons l'autorisation de scanner.

Pour l'installation de docker sur une machine linux nous allons faires les commandes suivantes : 

```bash
curl -fsSl https://get.docker.com/ | sh
```

Ici l'option -f permet de s'il y a une erreur que cela échoue silencieusement, ce qui veut dire qu'au lieu de renvoyer le code de la page erreur 404 cela va renvoyer qu'une simple erreur.

L'option -s permet de dire qu'on veut rester silencieux sur la barre de production et sur les stats du téléchargement. 

L'option -S "show error" cela permet que même en silencieux, cela nous renvoie le code d'erreur en cas de problème. 

L'option -L permet de suivre les redirections, sachant que Docker installe les paquets en faisant des redirections, il est donc indispensable.

Sh exécute le code directement.


Maintenant nous allons faire en sorte d'éviter de mettre sudo devant chaque ligne de commande Docker, pour cela on va faire : 

```bash
sudo usermod -aG docker $USER
```

Maintenant, nous allons vérifier notre version de Docker. Pour cela, nous allons faire : 

```bash
docker --version
```

Résultat : 

![](../Write%20Up/img/Pasted%20image%2020260716171912.png)

Maintenant nous allons essayer d'exécuter Hello World que vous connaissez si bien haha.

```bash
docker run hello-world
```

![](../Write%20Up/img/Pasted%20image%2020260716172119.png)

On peut voir que notre docker est donc fonctionnel, mais si ce n'est pas le cas, alors nous allons y remédier. 

Si vous avez ce problème tel que : 
![](../Write%20Up/img/Pasted%20image%2020260716172230.png)

C'est que notre utilisateur actuel n'a pas encore les permissions pour les sockets Docker.

Cela est dû car il n'y a pas encore de groupes pour Docker.

On va donc y remédier via : 

```bash
sudo usermod -aG docker $USER
```

Mais cela ne suffit pas puisque nous avons pas mis à jour on va donc régler cela en faisant : 

```bash 
newgrp docker
```

Maintenant on peut regarder si le service est actif. 

![](../Write%20Up/img/Pasted%20image%2020260716175835.png)

Si celui-ci est inactif vous pouvez lancer la commande suivante  : 

```bash
sudo systemctl enable --now docker
```

Maintenant on peut retester de faire : 

```bash
docker run hello-world
```

Vous devriez avoir ce résultat : 

![](../Write%20Up/img/Pasted%20image%2020260716180110.png)

Pour lancer le juicy-shop de OWASP nous allons lancer le docker via l'image qui est disponible:  

```bash
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
```

docker run va nous permettre de lancer le conteneur depuis une image et de la démarrer. 

-d permet de détacher le conteneur et de le faire tourner en arrière plan et donc de nous rendre la main sur le terminale.

-p 3000:3000 ici on fais un mapping de port de la forme : 
```bash 
port de notre machine:port du conteneur
```
--name pour donner un nom car sinon docker va donner un nom aléatoire et ce n'est pas ce que nous voulons.

et le dernier champs après le nom est l'image docker qu'on peut retrouver sur Docker Hub.


Bon, maintenant voyons voir ce que je fais plus concrètement. 


## Directory listing


La première option qui va être développée est plutôt simple, c'est un simple directory listing qui va récupérer les différents endpoints mis dans le fichier que l'utilisateur aura rentré. J'ai pris plusieurs nordiste que j'ai donc concaténer par exemple une wordlist sur les noms de fichier, etc... 

Le but ici est de récupérer simplement les endpoints sur le site cible qui renvoie un code HTTP 200. 

Ce qui signifie que le endpoint est accessible. 

Pour cela je vous invite à aller voir le script directory_listing.py qui est simple de compréhension.



## Scanneur de services.

On pourrait exécuter nmap automatiquement sur la cible qui a été rentrée. Mais cela n'est pas très intéressant, le but ici dans ce tools est de notamment refaire des outils qui sont déjà créés mais de soit les améliorer soit de les comprendre en les développant de manière à tout automatiser. 

Dans ce cas, nous allons utiliser la librairie Socket qui va nous permettre d'envoyer des paquets sur chacun des ports possibles sur une machine est en examinant le 3 way handshake qui est le principe TCP basique qui est Syn => Syn+Ack => Ack, ce qui veut dire, demande de connexion, connexion établie, puis connexion établie puis connexion fermé. 

Tout d'abord, pour la compréhension, nous utiliserons la librairie Socket. Qui lui va tenter d'aller jusqu'au bout de la connexion TCP, ce qui veut dire d'aller jusqu'au Ack final. Mais je vous le dis, cette méthode peut être facilement détectable puisque plusieurs outils de type SCADA peuvent détecter ou enregistrer ce qui se passe dès lors qu'il y a une connexion établie. Notre but ici est de simuler une blackbox et donc d'utiliser de manière telle qu'un attaquant le ferait dans la vraie vie, par exemple.


Donc premièrement nous allons faire avec les sockets de connexion ceci nous permettra de comprendre comment initié des connexions TCP mais nous basculeront rapidement vers un deuxième script qui utilisera la librairie scapy qui lui permet directement de formé des paquets et de les envoyers.

Pour comprendre scapy sont utilité ou autre je vous invite à lire : 
https://liora.io/scapy-tout-savoir

Mais aussi à vous référencé à leurs documentations : 

https://scapy.net

Ils faut savoir que nous ne pouvons pas simplement nous référencer au numéro de port etc pour les différentes services. En réalité nous allons devoir donc tout d'abord découvrir les ports qui sont ouvert, fermé ou filtré puis de lancer une connexion complète TCP pour pouvoir avoir la version du service et le service qui tourne sur le port.