
Tout d'abord, nous allons initialiser notre environnement, pour cela nous allons utiliser Docker qui va nous permettre, via une image, de créer un conteneur. Grâce à ce conteneur, nous allons pouvoir créer d'office plusieurs vulnérabilités et tenter de les scanner sans devoir être dans l'illégal en l'essayant sur des sites publics. Ici le projet restera strictement local ou sur des serveurs dont nous avons l'autorisation de scanner.

Pour l'installation de docker sur une machine linux nous allons faires les commandes suivantes : 

```bash
curl -fsSl https://get.docker.com/ | sh
```

Ici l'option -f permet de s'il y a une erreur que cela échoue silencieusement, ce qui veut dire qu'au lieu de renvoyer le code de la page erreur 404 cela va renvoyer qu'une simple erreur.

L'option -s permet de dire qu'on veut rester silencieux sur la barre de production et sur les stats du téléchargement. 

L'option -S "show error" cela permet que même en silencieux cela nous renvoie le code d'erreur en cas de problème. 

L'option -L permet de suivre les redirections, sachant que docker installe les paquets en faisant des redirection il est donc indispensable.

sh exécute le code directement.


Maintenant nous allons faire en sorte d'éviter de mettre sudo devant chaque ligne de commande docker, pour cela on va faire : 


```bash
sudo usermod -aG docker $USER
```
