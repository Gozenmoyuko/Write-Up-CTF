

Dans ce challenge notre but est d'obtenir accès à l'endpoint /web-serveur/ch63/admin .

Pour cela nous avons deux endpoint qui nous sont donnée avec les deux méthodes disponible les voici : 

![](Write%20Up/img/Pasted%20image%2020260411155122.png)


Au passage le code source nous ai donnée en dessous : 

`#!/usr/bin/env python3`
# `-*- coding: utf-8 -*-`
`from flask import Flask, request, jsonify`
`from flask_jwt_extended import JWTManager, jwt_required, create_access_token, decode_token`
`import datetime`
`#from apscheduler.schedulers.background import BackgroundScheduler`
`import threading`
`import jwt`
`from config import *`

# `Setup flask`
`app = Flask(__name__)`

`app.config['JWT_SECRET_KEY'] = SECRET`
`jwtmanager = JWTManager(app)`
`blacklist = set()`
`lock = threading.Lock()`

# `Free memory from expired tokens, as they are no longer useful`
`def delete_expired_tokens():`
    `with lock:`
        `to_remove = set()`
        `global blacklist`
        `for access_token in blacklist:`
            `try:`
                `jwt.decode(access_token, app.config['JWT_SECRET_KEY'],algorithm='HS256')`
            `except:`
                `to_remove.add(access_token)`
        
        `blacklist = blacklist.difference(to_remove)`

`@app.route("/web-serveur/ch63/")`
`def index():`
    `return "POST : /web-serveur/ch63/login <br>\nGET : /web-serveur/ch63/admin"`

# `Standard login endpoint`
`@app.route('/web-serveur/ch63/login', methods=['POST'])`
`def login():`
    `try:`
        `username = request.json.get('username', None)`
        `password = request.json.get('password', None)`
    `except:`
        `return jsonify({"msg":"""Bad request. Submit your login / pass as {"username":"admin","password":"admin"}"""}), 400`

    `if username != 'admin' or password != 'admin':`
        `return jsonify({"msg": "Bad username or password"}), 401`

    `access_token = create_access_token(identity=username,expires_delta=datetime.timedelta(minutes=3))`
    `ret = {`
        `'access_token': access_token,`
    `}`
    
    `with lock:`
        `blacklist.add(access_token)`

    `return jsonify(ret), 200`

# `Standard admin endpoint`
`@app.route('/web-serveur/ch63/admin', methods=['GET'])`
`@jwt_required`
`def protected():`
    `access_token = request.headers.get("Authorization").split()[1]`
    `with lock:`
        `if access_token in blacklist:`
            `return jsonify({"msg":"Token is revoked"})`
        `else:`
            `return jsonify({'Congratzzzz!!!_flag:': FLAG})`


`if __name__ == '__main__':`
    `scheduler = BackgroundScheduler()`
    `job = scheduler.add_job(delete_expired_tokens, 'interval', seconds=10)`
    `scheduler.start()`
    `app.run(debug=False, host='0.0.0.0', port=5000)`


Explication rapide du code : 

Lorsqu'on au départ nous avons juste les importations, on sait donc que le site utilise Flask et JWT le premier Setup est la configuration de flask pour le cookie JWT. Qui ont le rappel contient 3 champs qui sont les suivants 

- Header 
- Payload 
- Signature
Tout les trois séparé par un "." . 

Bon maintenant que nous savons cela essayons de voir comment réagit le site web qui nous ai donnée. 

On peut déjà savoir à peut près grâce au code source. Car lorsqu'on va lancer une requête GET vers la racine du site qui est http://challenge01.root-me.org/web-serveur/ch63/

Cela va nous renvoyer nos deux endpoint. Preuve : 

![](../img/Pasted%20image%2020260411160354.png)

Maintenant que nous savons cela essayons de voir comment ce comporte le endpoint par la méthode POST de /web-serveur/ch63/login


![](../img/Pasted%20image%2020260411160955.png)

Nous pouvons voir que le serveur nous envoie un message d'erreur nous disant que notre requête est mauvaise, et qu'il faut mettre les données d'utilisateurs suivant : 

{"username":"admin","password":"admin} 

Sachant cela on va pouvoir tenter de ce connecter au compte qui détient c'est donnée. 
Mais avant cela il faut rajouter en Header HTTP : Content-Type : application/json

Cette header va nous permettre d'échanger des données JSON(Javascript Object Notation) avec le serveur. 

Voici à quoi doit ressemblé la requête : 

![](../img/Pasted%20image%2020260411161321.png)


Et voici la réponse : 
![](../../Pasted%20image%2020260411161401.png)

Un token JWT vient d'être générer et nous à été donnée. On peut remarqué que c'est bien un jeton JWT puisqu'il détient les 3 champs que je vous ai rappelez juste avant. 

