# Authentification-serveur-apache-avec-kerberos
Dans ce référentiel, j'ai tenté de mettre en place une authentification du serveur apache graçe au protocole "Kerberos".

# Configuration de Kerberos dur Ubuntu
### Kerberos ### est un protocole d’authentification qui prend en charge le concept d’authentification unique (SSO). Après s’être authentifiés une fois au début d’une session, les utilisateurs peuvent accéder aux services réseau dans un domaine Kerberos sans s’authentifier à nouveau. Pour que cela fonctionne, il est nécessaire d’utiliser des protocoles réseau compatibles Kerberos.
Dans le cas de HTTP, la prise en charge de Kerberos est généralement fournie à l’aide du mécanisme d’authentification SPNEGO (Simple and Protected GSS-API Negotiation). Ceci est également connu sous le nom d'«authentification intégrée » ou « authentification de négociation ». Apache ne supporte pas SPNEGO lui-même, mais le support peut être ajouté au moyen du module d’authentification.`mod_auth_kerb`
