# Sharedflow gcp-token

## Description

This sharedflow will make it possible to request an OAuth token from Google according to the service account that was passed to it as a parameter.
Once the token has been retrieved, then it is cached for the duration defined in the configuration file _caches.json_ (may be different for each environment. In our examples this value is 10 seconds in dev, 60 seconds in test and 3500 seconds in prd).
[Documentation Google](https://developers.google.com/identity/protocols/oauth2/service-account)

## Pré-requis
Lors de l'utilisation de ce sharedflow dans un proxy il faut d'abord prévoir :

- A service account that has the rights to call an API on GCP via IAP
- The JSON of this service account
- Create an entry in the encrypted KVM *gcp-service-account-credentials* with the project name as key and the service account JSON content as value
- Create in the proxy a variable named *private.credentialsjson* whose value is the content of the JSON of the service account

## How to

This sharedflow executes 7 policies in a row :
- lc.gcp-token
- js.extract-credentials
- gjwt.generate-JWT
- sc.gcp-oauth
- ev.extract-json
- am.add-authorization
- pc.gcp-token

### lc.gcp-token

This policy checks if a value exists in a cache whose name is composed :
- Organization name
- Environment name
- Proxy name
- Revision number
- Target name
If a value is found then we instantiate the value of the _google-credentials.access-token_ variable with this value.
This avoids asking for a new token if this value is still present (cache duration still valid).

** The next 4 steps are only executed if the _google-credentials.access-token_ variable is not empty. **

### js.extract-credentials

This policy extracts each key/value pair from the *private.credentialsjson* variable (which contains the JSON content of the service account).

### gjwt.generate-JWT

This policy uses the retrieved information to generate a JWT token. This token is written in the variable *output_jwt*.

### sc.gcp-oauth

This policy asks GCP to create an OAuth token from the previously generated JWT token. This token is written in the variable *callout-token*.

### ev.extract-json

This policy extracts the OAuth token returned earlier and creates a variable *google-credentials.access-token*.

### am.add-authorization

This policy creates an *Authorization* header whose value is *Bearer * + the content of the _google-credentials.access-token_ variable.

### pc.gcp-token

This policy caches the value of the _google-credentials.access-token_ variable in a cache whose name is composed :
- Organization name
- Environment name
- Proxy name
- Revision number
- Target name