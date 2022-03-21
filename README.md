# APIGEE - SECURE GCP BACKEND WITH OAUTH OR IAP

[![Apigee](https://upload.wikimedia.org/wikipedia/commons/a/aa/Apigee_logo.svg)](https://cloud.google.com/apigee)

This is a code to deploy sharedflows which are used when your backend is secured by GCP OAuth (via Service Account) or IAP (via Service Account).

## Prerequisite

- A GCP Project with, for example, an App Engine.
- A service Account with good rights to call your App Engine and his JSON Key.
- An Apigee account with an organization (free or paid).
- [Maven](https://github.com/apigee/apigee-deploy-maven-plugin) or [apigeetools](https://github.com/apigee/apigeetool-node) install on your computer to deploy the sharedflow.
- Maven install on local computer.

### Installation

- Create a new encrypted Key Value Map on your Apigee Organisation which name is _gcp-service-account-credentials_.
- In this KVM, add a new key/value pair with a key name _credentials_ and for value the JSON Key value.
- Update cache files with good timeout in seconds for each environment :
    - [Dev](sharedflows/gcp-token-oauth/env/dev/caches.json)
    - [Test](sharedflows/gcp-token-oauth/env/test/caches.json)
    - [Prd](sharedflows/gcp-token-oauth/env/prd/caches.json)

## Deploy

This repository contain the maven configuration to deploy this sharedflow on 3 environments : dev, test or prd.

How to use it :
```
cd sharedflows/gcp-token-oauth
mvn install -P=dev -Dusername=XXXX -Dpassword=YYYY -Dorganization=ZZZZ
```

## Versions

**Last version :** 1.0

## Auteurs

* **Thomas** _alias_ [@tonyglandyl](https://github.com/tonyglandyl28)

