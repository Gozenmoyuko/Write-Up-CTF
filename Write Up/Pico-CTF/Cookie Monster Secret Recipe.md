

Voyons voir l'énoncer. 

![Pasted image 20250327195408.png](../img/Pasted%20image%2020260307165931.png)

Okay nous avons un utilisateur ou quelqu'un qui à cacher sont mot de passe quelques parts et qu'il faut le récupérer et sûrement le déchiffre. 

Bon c'est partie, let's go lancer le site. 

![../img/Pasted image 20260307170210.png](../img/Pasted%20image%2020260307170210.png)

On voit que c'est un login basique, essayons test en username et test en password.


![../img/Pasted image 20260307170829.png](../img/Pasted%20image%2020260307170829.png)

Tiens tiens tiens, il n'accepte que les cookies. Bon récupérons avec Burpsuite. 

![../img/Pasted image 20260307170959.png](../img/Pasted%20image%2020260307170959.png)

Cliquez sur les deux endroits, un navigateur va s'ouvrir. 

![../img/Pasted image 20260307171039.png](../img/Pasted%20image%2020260307171039.png)

Mettez le lien du CTF en haut. 

Okay on à récupérer la requête cliquez dessus et faite CTRL+R cela va l'envoyer au Repeater.

![../img/Pasted image 20260307171228.png](../img/Pasted%20image%2020260307171228.png)


![../img/Pasted image 20260307171305.png](../img/Pasted%20image%2020260307171305.png)

Cliquez sur Send et vous allez avoir la réponse du site comme moi. 

Essayons d'injecter l'username et le password en JSON comme ceci : 

![../img/Pasted image 20260307171700.png](../img/Pasted%20image%2020260307171700.png)

Okay cliquez sur Send et voilà vous verrez que cela ne marche pas. Pourquoi ? car on ne dépose pas actuellement c'est information avec le login.php regarder bien en haut à gauche on est dans la racine du site "/" cela peut-être index.html on ne sais pas, refaisons avec login.php.


![../img/Pasted image 20260307171949.png](../img/Pasted%20image%2020260307171700.png)

Bingo on à le cookie, vous pouvez selectionner le cookie et regarder le Inpesctor qui va vous le décoder de base64 en ce qui est lisible.

![../img/Pasted image 20260307172101.png](../img/Pasted%20image%2020260307172101.png)

Bingo.

