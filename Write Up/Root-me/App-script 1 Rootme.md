
Étape 1 : 

![Pasted image 20250327195408.png](../img/Pasted%20image%2020250327195408.png)
- Ici nous voyons que nous n'avons pas la permission d'exécuter le fichier  ".passwd" avec /executable Donc pour voir les fichiers auxquels on avait accès, nous avons utilisés "ls -la" (ls qui sert à listez les fichiers et le -l permet d'affichier un détaille et -a permet de affichier les fichiers cachées on peut les additionnées en -la).
- Par la suite on voit que le fichier .passwd est dans nos fichier accessible et cacher.
---
Étape 2 : 
![Pasted image 20250327195527.png](../img/Pasted%20image%2020250327195527.png)
Dans cette étape, on regarde si on peut donc lire directement notre fichier .passwd et bien sûr que non, sinon ce ne serait pas intéressant :). 

C'est pourquoi on va se poser la question "comment cacher la commande cat dans la commande ls" et on regardera si ça marche et si notre intuition est bonne.

---
Étape 3:
On va utiliser les valeurs binaires stockées dans cat et ls et on cache le binaire de cat dans le binaire de ls. Pour cela, on utilise la commande which qui regarde où elles sont situées dans le PATH.

Dans le cas de cat :
![Pasted image 20250327200438.png](../img/Pasted%20image%2020250327200438.png)

Dans le cas de ls :
![Pasted image 20250327201339.png](../img/Pasted%20image%2020250327201339.png)


Par la suite on regarde où est stocké notre PATH dans l'environnement : 
pour cela on utilise la commande ```print``` suivie de ```env``` qui est notre environnement ce qui va nous donner ```printenv``` 
![Pasted image 20250327201547.png](../img/Pasted%20image%2020250327201547.png)

Ici on voit PATH = + un chemin d'accès où sont stockées nos variables d'environnement. 

---

Étape 4 : 

On va se situer dans notre fichier /tmp pour pouvoir faire notre manipulation sans modifier notre environnement total, ce qui va nous donner. 

![Pasted image 20250327201916.png](../img/Pasted%20image%2020250327201916.png)
Puis nous allons créer un fichier dans ce /tmp pour pouvoir faire nos manipulations nécessaires, qui est celui de copier notre valeur binaire de cat dans notre commande ls. On utilisera ```mkdir + nom du fichier```

![Pasted image 20250327202325.png](../img/Pasted%20image%2020250327202325.png)
puis on se déplace dans ce même fichier encore avec cd : 

![Pasted image 20250327202407.png](../img/Pasted%20image%2020250327202407.png)
puis on utilisera echo $PATH en bash. Pour une variable on utilise la systaxe $ devant la variable :

![Pasted image 20250327202608.png](../img/Pasted%20image%2020250327202608.png)

On utilisera export puis on indiquera PATH pour et on lui donnera la valeur qui est stockée dans notre variable  ```$PATH```  qui est située dans notre /tmp/gozen puis on ajoute : pour indiquer la variable qu'on va stocker soit la commande ```/tmp/gozen:$PATH``` 
```export PATH="ton répertoire":$PATH```


---

Étape 5 : 

![Pasted image 20250327210410.png](Pasted%20image%2020250327210410.png%7C.md)


On copie la valeur binaire de cat dans fonction ls pour cacher notre fonction cat et donc utiliser le cat à travers ls qui lui est autorisé à être utilisée.

puis on sort de notre dossier à la fin de toutes ses manipulations en utilisant ```cd ```. On retourne donc dans 
![Pasted image 20250327210904.png](../img/Pasted%20image%2020250327210904.png)**
on réessaie 'ls' mais on voit que cela ne marche pas. :(

Pour cela dernière étape on exécute le fichier /ch11 en faisant ./ch11 
![Pasted image 20250327210956.png](../img/Pasted%20image%2020250327210956.png)

Et voilà le problème résolu, on a le mot de passe demandé ;).