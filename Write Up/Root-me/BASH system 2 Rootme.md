**Contexte** 

![Pasted image 20250407121510.png](../img/Pasted%20image%2020250407121510.png)
Ici nous voyons lorsque qu'on se connecte au ctf (SSH) que nous somme l'utilisateur **app-script-ch12@challenge02**


![Pasted image 20250407121657.png](../img/Pasted%20image%2020250407121657.png)
on regarde qu'elle sont les fichiers cachées dans notre serveur ssh grâce a la fonction **ls** qui permet de lister **-la** pour lister les fichiers cacher

![Pasted image 20250407121814.png](../img/Pasted%20image%2020250407121814.png)
on essaye ici d'exécuter le binaire du code et nous voyons que nous avons pas la permission 

---
Étape 1 : Essayons donc avec cat de voir si nous pouvons au moins lire ce fichier.

![Pasted image 20250407121943.png](../img/Pasted%20image%2020250407121943.png)
Ici nous voyons que nous pouvons bel est bien lire le fichier qui est un script d'intéraction avec notre dossier ch12.

Il appelle `system("ls -lA /challenge/app-script/ch12/.passwd")`, donc il exécute la commande `ls` **via le shell**.

L'objectif est de lire le contenu de `.passwd`, ce fichier appartenant à l’autre utilisateur (`app-script-ch12-cracked`) :

---
**Comment ont vas faire ?** 

Puisque le C appelle `system("ls -lA ...")`, et que **`system()` utilise `$PATH` pour chercher `ls`**, on peux créer **notre propre script `ls` malveillant** et modifier notre `PATH` pour qu’il soit appelé à la place du vrai.

 ___
 **Étape 2: 
  ![Pasted image 20250407132810.png](../img/Pasted%20image%2020250407132810.png)
 Ici echo affiche le texte de statut dans un fichier ou sur notre écran 
 le -e permet de prendre en compte le retour à la ligne qui est ici ``\n`` 

par la suite on identifie ce que nous allons mettre dans notre configuration bash ici notre bash va être configurer de façon à être en première ligne qu'on identifie ici par #! puis la suite sera le contenu de notre première ligne``` #!/bin/bash``` puis on pourrais ce dire 'mais le \ncat va être sur la première ligne' mais nan puisque notre **\n** permet de sauté la ligne ici notre bash va comprendre comme si nous exécutions une commande mais va incorporer notre deuxième ligne dans notre commande on à donc en deuxième ligne: 
```cat /challenge/app-script/ch12/.passwd``` puis on incorpore ce bash dans notre nouveau ls qui va contenir notre bash ici on prend ls car le script utilise la fonction ls.
___
Étape 3: 

![Pasted image 20250407134113.png](../img/Pasted%20image%2020250407134113.png)
ici chmod permet de modifier les permission d'un fichier ici on modifie les permission de **/tmp/gozen/ls**  
Le `+x` indique que l'on ajoute la permission d'exécution (execute permission) au fichier pour les utilisateurs.

___
![Pasted image 20250407134309.png](../img/Pasted%20image%2020250407134309.png)
Étape 4: 
	Puisque le script appel system pour  vérifier les fonction qui sont utiliser en utilisant PATH on va donc exporter notre PATH dans notre nouvelle variable PATH qui est stocker dans notre dossier /tmp/gozen.
	 
Lorsque nous allons essayer d'exécuter le binaire de ch12 ce qui va se passé c'est que le script va appeler ls qui est modifier car on a remplacer ça valeur binaire par celle de cat avec echo - e puis en donnant les permission à cette nouvelle fonction ls grâce à chmod -x.
	Puis il va vérifier si c'est la fonction ls qui est utiliser il va donc appeler PATH pour voir si c'est bien ls qui est utiliser mais puisqu'on a changer d'endroit ou se trouve PATH par $PATH qui est dans notre /tmp/gozen est bien il va appeler ls de notre /tmp/gozen est pas celui de /bin/ls et il va avoir la valeur de ```cat /challenge/app-script/ch12/.passwd.
Et voici comment on peu réexécuter notre dossier ch12 et le .passwd
