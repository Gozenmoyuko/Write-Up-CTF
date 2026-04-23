
Dans ce challenge il faudra tout d'abord trouver un fichier backup pour que par la suite on puisse récupérer le contenu du index.php 

Je vous rappel tout les extensions de backup possible : 
https://www.filemagic.com/en/type/backup-files/page/1/


![](../img/Pasted%20image%2020260423220905.png)

Voyons voir comment est le site : 

![](../img/Pasted%20image%2020260423221044.png)

Ici on peut voir qu'en mettant un mot de passe tel que test cela nous renvoie "try again"

![](../img/Pasted%20image%2020260423221409.png)

Pour ce qui est du fichier backup j'ai décider de regarder lesquels étaient le plus courant. Pour vérifier je vais taper dans l'url du site la chose suivante : 

http://challenge01.root-me.org/web-serveur/ch17/index.php[l'extension qu'on veut tester]

http://challenge01.root-me.org/web-serveur/ch17/index.php.bak
http://challenge01.root-me.org/web-serveur/ch17/index.php.zip
http://challenge01.root-me.org/web-serveur/ch17/index.php.tar
http://challenge01.root-me.org/web-serveur/ch17/index.php.sql
http://challenge01.root-me.org/web-serveur/ch17/index.php.daf
etc... 

Mais heureusement que Burpsuite existe ;) 

Je vais vous montrer comment faire avec Burpsuite : 

![](../img/Pasted%20image%2020260423222127.png)

Cela vas vous ouvrir Chromonium, par la cliquez sur intercept Off vous devriez vois que cela ce met en intercept on : 

![](../img/Pasted%20image%2020260423222232.png)

Taper l'url sur chromonium : 


![](../img/Pasted%20image%2020260423222309.png)


Vous devriez voir cela sur Burpsuite : 
![](../img/Pasted%20image%2020260423222344.png)


Même si je suis un grand lover de l'option Repeater cette fois-ci on va utiliser l'option Intruder, faite clique droit sur la requête puis faite "Send to Intruder" ou soit petit tips cliquez dessus et faite CTRL + I. 

![](../img/Pasted%20image%2020260423222450.png)

Vous allez avoir cela. Maintenant comment faire ? 

On va ajouter en cliquant sur Add l'endroit ou burpsuite va bruteforce nos extension de backup mais avant ça il faut rajouter un "." après le index.php pour que lorsqu'il va bruteforce il fasse : 

index.php.zip 
index.php.back
index.php.bak 
etc.. 
si on met pas notre "." cela donnera 
index.phpzip 

ce n'est pas ce qu'on veut : 

Première étape rajouter le "." : 

![](../img/Pasted%20image%2020260423222725.png)

Par la suite on va rajouter la positions ou va Burpsuite va faire le bruteforce : 

![](../img/Pasted%20image%2020260423222913.png)

Mettez vous après le . que l'ont viens de rajouter puis cliquez en haut sur Add 

Une nouvelle page à droite viens d'être créé. 

Pour le premier champs on laisse ce qui est mis ce qui veut dire Simple list etc...

Pour le champs Payload configuration cliquez sur Add from list : 
![](../img/Pasted%20image%2020260423223213.png)

Et choisissez l'option "Extensions Short" vous devriez avoir cela : 

![](../img/Pasted%20image%2020260423223315.png)

On peut reconnaitre notre extension backup etc... 

Maintenant cliquez sur Start Attack en haut : 

![](../img/Pasted%20image%2020260423223405.png)

Résultat : 

![](../img/Pasted%20image%2020260423223505.png)

Bingo on voit qu'un à été trouver qui est le .bak 

Maintenant deux choix s'offre à vous, soit vous regardez le code sur la réponse directement avec burpsuite, soit vous allez sur votre navigateur et vous tapez : 
http://challenge01.root-me.org/web-serveur/ch17/index.php.bak 

Cela va vous installer le fichier. 

Maintenant analysons le code : 

```php
<?php


function auth($password, $hidden_password){
    $res=0;
    if (isset($password) && $password!=""){
        if ( $password == $hidden_password ){
            $res=1;
        }
    }
    $_SESSION["logged"]=$res;
    return $res;
}



function display($res){
    $aff= '
	  <html>
	  <head>
	  </head>
	  <body>
	    <h1>Authentication v 0.05</h1>
	    <form action="" method="POST">
	      Password&nbsp;<br/>
	      <input type="password" name="password" /><br/><br/>
	      <br/><br/>
	      <input type="submit" value="connect" /><br/><br/>
	    </form>
	    <h3>'.htmlentities($res).'</h3>
	  </body>
	  </html>';
    return $aff;
}



session_start();
if ( ! isset($_SESSION["logged"]) )
    $_SESSION["logged"]=0;

$aff="";
include("config.inc.php");

if (isset($_POST["password"]))
    $password = $_POST["password"];

if (!ini_get('register_globals')) {
    $superglobals = array($_SERVER, $_ENV,$_FILES, $_COOKIE, $_POST, $_GET);
    if (isset($_SESSION)) {
        array_unshift($superglobals, $_SESSION);
    }
    foreach ($superglobals as $superglobal) {
        extract($superglobal, 0 );
    }
}

if (( isset ($password) && $password!="" && auth($password,$hidden_password)==1) || (is_array($_SESSION) && $_SESSION["logged"]==1 ) ){
    $aff=display("well done, you can validate with the password : $hidden_password");
} else {
    $aff=display("try again");
}

echo $aff;

?>
```


On peut voir plusieurs choses intéressante mais surtout en bas dans $aff qui nous renvoie la valeur du password que si une des deux conditions est valider. 

La première est que le password taper par l'utilisateur est la même que le flag (ce qui est logique puisque cela veut dire qu'il a mis le bon mot de passe ce qui veut dire que les deux donne 1) et l'autre c'est que si la SESSION de l'utilisateur est égale à 1 alors cela veut dire qu'il est connecter (ce qui est logique aussi car la Session est active) mais surtout il y a deux failles dans ce code, la première est que dans le register_globals est actif est que celui-ci accepte les différentes variable de chaines de requête comme 
```php
$_GET 
$_SESSION 
```
la deuxième est que la conditions pour valider le challenge est un "ou" || est pas un "Et" &&. 

Donc il suffit qu'un des deux soit valide. Pour cela c'est très simple puisque la méthode GET est valable il suffit  de mettre une des conditions à l'interieur sachant que le $ veut dire "variable" c'est des variables prédéfini(`_GET, _SESSION etc...`) mais puisqu'on va le mettre dans l'url on l'enlèvera, et aussi on prend la condition : 

```php
_SESSION["logged"]==1
```

Il faut savoir qu'en PHP lorsqu'il y a un seul guillemet cela signifie que c'est une chaîne littérale or les doubles guillemets peuvent contenir une variable à l'intérieur. Voyez cela un peut comme le Fstring en Python.

Donc ici logged n'est pas une chaîne de caractère mais une variable que l'ont peut retrouver plusieurs fois dans le code. Il faut donc enlever c'est double guillemet pour que notre requête soit bonne car on va faire une requête HTTP et non via PHP donc il faut l'enlever. Il faudra aussi ne mettre qu'un seul = puisque le double est propre à PHP et pas HTTP

Ce qui donne : 

```http
http://challenge01.root-me.org/web-serveur/ch17/?_SESSION[logged]=1
```


![](../img/Pasted%20image%2020260423230327.png)

Bingo !! 
