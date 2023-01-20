# Authentification-serveur-apache-avec-kerberos
Dans ce référentiel, j'ai tenté de mettre en place une authentification du serveur apache graçe au protocole "Kerberos".

# Configuration de Kerberos dur Ubuntu
Kerberos est un protocole d’authentification qui prend en charge le concept d’authentification unique (SSO). Après s’être authentifiés une fois au début d’une session, les utilisateurs peuvent accéder aux services réseau dans un domaine Kerberos sans s’authentifier à nouveau. Pour que cela fonctionne, il est nécessaire d’utiliser des protocoles réseau compatibles Kerberos.
Dans le cas de HTTP, la prise en charge de Kerberos est généralement fournie à l’aide du mécanisme d’authentification SPNEGO (Simple and Protected GSS-API Negotiation). Ceci est également connu sous le nom d'«authentification intégrée » ou « authentification de négociation ». Apache ne supporte pas SPNEGO lui-même, mais le support peut être ajouté au moyen du module d’authentification.`mod_auth_kerb`

![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)

!! Étant donné que le protocole Kerberos implique un horodatage, les horloges des trois machines doivent être synchronisées.


# Réference et  Inspiration
Ne sachant pas vraiment par ou commencer, je me suis inspirer de:
 * Pour la présentation sur github
    * [Yorsa270](https://github.com/yosra270/postgresql-auth-with-kerberos/)
 
 * Mr Souheib de la chaine Techwall pour l'introduction à kerberos:
    * [Théorique](https://youtu.be/DxlzvDNgkFg)
    * [Pratique](https://youtu.be/vx2vIA2Ym14)

* Installation et configuration et mise en place d'un server web Apache2:
    * [Grafikart](https://youtu.be/arVwa7jvp5M))
