
Tout d'abord, nous allons initialiser notre environnement, pour cela nous allons utiliser Docker qui va nous permettre, via une image, de créer un conteneur. Grâce à ce conteneur, nous allons pouvoir créer d'office plusieurs vulnérabilités et tenter de les scanner sans devoir être dans l'illégal en l'essayant sur des sites publics. Ici le projet restera strictement local ou sur des serveurs dont nous avons l'autorisation de scanner.

Pour l'installation de docker sur une machine linux nous allons faires les commandes suivantes : 

```bash
curl -fsSl https://get.docker.com/ | sh
```