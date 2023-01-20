# Authentification-serveur-apache-avec-kerberos
Dans ce référentiel, nous avons tentés de mettre en place une authentification du serveur web apache graçe au protocole **Kerberos**.

# Configuration de Kerberos dur Ubuntu
**Kerberos** est un protocole d’authentification qui prend en charge le concept d’authentification unique (SSO). Après s’être authentifiés une fois au début d’une session, les utilisateurs peuvent accéder aux services réseau dans un domaine Kerberos sans s’authentifier à nouveau. Pour que cela fonctionne, il est nécessaire d’utiliser des protocoles réseau compatibles Kerberos.
Dans le cas de HTTP, la prise en charge de Kerberos est généralement fournie à l’aide du mécanisme d’authentification **SPNEGO** (Simple and Protected GSS-API Negotiation). Ceci est également connu sous le nom d'«authentification intégrée » ou « authentification de négociation ». Apache ne supporte pas SPNEGO lui-même, mais le support peut être ajouté au moyen du module d’authentification. `mod_auth_kerb`

![Kerberos en Image](https://github.com/pra-tek/Authentification-serveur-apache-avec-kerberos/blob/main/Capture%20d'%C3%A9cran/Kerberos-Protocol.jpg)


# Hôtes et adresse ip

Nous n'avons pas nécéssairement besoin de trois machines pour celà. Une machine peut tout à fait contennir deux rôles ( **KDC** et **server**). Mais dans notre cas, nous avons utilisé trois machines virtuelles sur [VmWare Workstation Pro](https://www.vmware.com/fr/products/workstation-pro/workstation-pro-evaluation.html/). Nos machines étant soit une distribution basé sur ubuntu, soit ubuntu.
Nos machines étant toutes les trois virtuelles, Nous n'avons pas eu besoin de modifier l'adaptateur réseau par defaut (** NAT **) pour les attribuer des addresses ip; Vm Ware s'en est chargé.

Nous pouvons vérifier les adresses IP des trois machines en les exécutant dans chacune d’elles.hostname -I

Dans notre cas :

* L’adresse IP de l’ordinateur client (machine virtuelle) est: **192.168.111.130**
* L’adresse IP de la machine du serveur web (machines virtuelles) est: **192.168.111.134**
* L’adresse IP de la machine KDC (machine virtuelle) est: **192.168.111.133**

Définissons des noms d’hôte pour chaque machine :

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/1.png)

![kdc](asset/Capture%20d'%C3%A9cran/Kdc/1.png)

![client](https://github.com/pra-tek/Authentification-serveur-apache-avec-kerberos/blob/main/Capture%20d'%C3%A9cran/Client/1.png)

!! Étant donné que le protocole Kerberos implique un horodatage, les horloges des trois machines doivent être synchronisées.

![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)

![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)



# Réference et  Inspiration
Ne sachant pas vraiment par ou commencer, je me suis inspirer de:
 * Pour la présentation sur github
    * [Yorsa270](https://github.com/yosra270/postgresql-auth-with-kerberos/)
 
 * Mr Souheib de la chaine Techwall pour l'introduction à kerberos:
    * [Théorique](https://youtu.be/DxlzvDNgkFg)
    * [Pratique](https://youtu.be/vx2vIA2Ym14)

* Installation et configuration et mise en place d'un server web Apache2:
    * [Grafikart](https://youtu.be/arVwa7jvp5M))
