# Sharedflow gcp-token

## Description

Ce sharedflow va permettre de demander un jeton OAuth à Google en fonction du service account qui lui ai passé en paramètre.
Une fois le token récupéré alors on le met en cache durant la durée définit dans le fichier de configuration _caches.json_ (peut être différent pour chaque environnement. Dans nos exemples cette valeur est de 10 seconde en dev, 60 secondes en test et 3500 secondes en prd).
[Documentation Google](https://developers.google.com/identity/protocols/oauth2/service-account)

## Pré-requis
Lors de l'utilisation de ce sharedflow dans un proxy il faut d'abord prévoir :

- Un sevice account qui a les droit d'appeler une API sur GCP via un IAP.
- Le JSON de ce service account.
- Créer une entrée dans le KVM chiffré *gcp-service-account-credentials* avec en clé le nom du projet et en valeur le contenu du JSON du service account.
- Créer dans le proxy une variable nommée *private.credentialsjson* dont la valeur est le contenu du JSON du service account.

## Fonctionnement

Ce sharedflow exécute 7 policies d'affilées :
- lc.gcp-token
- js.extract-credentials
- gjwt.generate-JWT
- sc.gcp-oauth
- ev.extract-json
- am.add-authorization
- pc.gcp-token

### lc.gcp-token

Cette policy vérifie si une valeur existe dans un cache dont le nom est composé :
- nom de l'organisation
- nom de l'environnement
- nom du proxy
- numéro de la révision
- nom de la target
Si une valeur est trouvée alors on instancie la valeur de la variable _google-credentials.id-token_ avec cette valeur.
Cela permet d'éviter de demander un nouveau token si cette valeur est toujours présente (durée de cache encore valide).

** Les 4 étapes suivantes ne sont exécutées que si la variable _google-credentials.id-token_ n'est pas vide.**

### js.extract-credentials

Cette policy permet d'extraire chaque paire clé/valeur de la variable *private.credentialsjson* (qui contient le contenu du JSON du service account).

### gjwt.generate-JWT

Cette policy utilise les informations récupérées pour générer un jeton JWT. Ce jeton est inscrit dans la variable *output_jwt*.

### sc.gcp-oauth

Cette policy demande à GCP la création d'un jeton OAuth à partir du jeton JWT généré précédemment. Ce jeton est inscrit dans la variable *callout-token*.

### ev.extract-json

Cette policy extrait le jeton OAuth renvoyé précédemment et crée une variable *google-credentials.id-token*.

### am.add-authorization

Cette policy crée une en-tête *Authorization* dont la valeur est *Bearer * + le contenu de la variable _google-credentials.id-token_.

### pc.gcp-token

Cette policy met en cache la valeur de la variable _google-credentials.id-token_ dans un cache dont le nom est composé :
- nom de l'organisation
- nom de l'environnement
- nom du proxy
- numéro de la révision
- nom de la target