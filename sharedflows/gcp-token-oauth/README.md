# Sharedflow gcp-token

## Description

Ce sharedflow va permettre de demander un jeton OAuth à Google en fonction du service account qui lui ai passé en paramètre.
[Documentation Google](https://developers.google.com/identity/protocols/oauth2/service-account)

## Pré-requis
Lors de l'utilisation de ce sharedflow dans un proxy il faut d'abord prévoir :

- Un sevice account qui a les droit d'appeler une API sur GCP
- Le JSON de ce service account
- Créer une entrée dans le KVM chiffré *gcp-service-account-credentials* avec en clé le nom du projet et en valeur le contenu du JSON du service account
- Créer dans le proxy une variable nommée *private.credentialsjson* dont la valeur est le contenu du JSON du service account

## Fonctionnement

Ce sharedflow exécute 4 policies d'affilées :
- js.extract-credentials
- gjwt.generate-JWT
- sc.gcp-oauth
- ev.extract-json

### js.extract-credentials

Cette policy permet d'extraire chaque paire clé/valeur de la variable *private.credentialsjson* (qui contient le contenu du JSON du service account).

### gjwt.generate-JWT

Cette policy utilise les informations récupérées pour générer un jeton JWT. Ce jeton est inscrit dans la variable *output_jwt*.

### sc.gcp-oauth

Cette policy demande à GCP la création d'un jeton OAuth à partir du jeton JWT généré précédemment. Ce jeton est inscrit dans la variable *callout-token*.

### ev.extract-json

Cette policy extrait le jeton OAuth renvoyé précédemment et crée une variable *google-credentials.access-token*.