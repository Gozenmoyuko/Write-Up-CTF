

Voyons voir l'énoncer. 

![[Write Up/img/Pasted image 20260307165931.png]]

Okay nous avons un utilisateur ou quelqu'un qui à cacher sont mot de passe quelques parts et qu'il faut le récupérer et sûrement le déchiffre. 

Bon c'est partie, let's go lancer le site. 

![[Write Up/img/Pasted image 20260307170210.png]]

On voit que c'est un login basique, essayons test en username et test en password.


![[Write Up/img/Pasted image 20260307170829.png]]

Tiens tiens tiens, il n'accepte que les cookies. Bon récupérons avec Burpsuite. 

![[Write Up/img/Pasted image 20260307170959.png]]

Cliquez sur les deux endroits, un navigateur va s'ouvrir. 

![[Write Up/img/Pasted image 20260307171039.png]]

Mettez le lien du CTF en haut. 

Okay on à récupérer la requête cliquez dessus et faite CTRL+R cela va l'envoyer au Repeater.

![[Write Up/img/Pasted image 20260307171228.png]]


![[Write Up/img/Pasted image 20260307171305.png]]

Cliquez sur Send et vous allez avoir la réponse du site comme moi. 

Essayons d'injecter l'username et le password en JSON comme ceci : 

![[Write Up/img/Pasted image 20260307171700.png]]

Okay cliquez sur Send et voilà vous verrez que cela ne marche pas. Pourquoi ? car on ne dépose pas actuellement c'est information avec le login.php regarder bien en haut à gauche on est dans la racine du site "/" cela peut-être index.html on ne sais pas, refaisons avec login.php.


![[Write Up/img/Pasted image 20260307171949.png]]

Bingo on à le cookie, vous pouvez selectionner le cookie et regarder le Inpesctor qui va vous le décoder de base64 en ce qui est lisible.

![[Write Up/img/Pasted image 20260307172101.png]]

Bingo.

