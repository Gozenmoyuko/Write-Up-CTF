
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

![](../Pasted%20image%2020260716180110.png)

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