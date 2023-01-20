# Authentification-serveur-apache-avec-kerberos
Dans ce référentiel, nous avons tentés de mettre en place une authentification du serveur web apache graçe au protocole **Kerberos**.

# Configuration de Kerberos dur Ubuntu
[**Kerberos**](https://fr.scribd.com/presentation/205732355/Kerberos) est un protocole d’authentification qui prend en charge le concept d’authentification unique (SSO). Après s’être authentifiés une fois au début d’une session, les utilisateurs peuvent accéder aux services réseau dans un domaine Kerberos sans s’authentifier à nouveau. Pour que cela fonctionne, il est nécessaire d’utiliser des protocoles réseau compatibles Kerberos.
Dans le cas de HTTP, la prise en charge de Kerberos est généralement fournie à l’aide du mécanisme d’authentification **SPNEGO** (Simple and Protected GSS-API Negotiation). Ceci est également connu sous le nom d'«authentification intégrée » ou « authentification de négociation ». Apache ne supporte pas SPNEGO lui-même, mais le support peut être ajouté au moyen du module d’authentification. `mod_auth_kerb`

![Kerberos en Image](Capture%20d'%C3%A9cran/Kerberos-Protocol.jpg)


# Hôtes et adresse ip

Nous n'avons pas nécéssairement besoin de trois machines pour celà. Une machine peut tout à fait contennir deux rôles ( **KDC** et **server**). Mais dans notre cas, nous avons utilisé trois machines virtuelles sur [VmWare Workstation Pro](https://www.vmware.com/fr/products/workstation-pro/workstation-pro-evaluation.html/). Nos machines étant soit une distribution basé sur ubuntu, soit ubuntu.
Nos machines étant toutes les trois virtuelles, Nous n'avons pas eu besoin de modifier l'adaptateur réseau par defaut (**NAT**) pour les attribuer des addresses ip; Vm Ware s'en est chargé.

Nous pouvons vérifier les adresses IP des trois machines en les exécutant dans chacune d’elles.hostname -I

Dans notre cas :

* L’adresse IP de l’ordinateur client (machine virtuelle) est: **192.168.111.130**
* L’adresse IP de la machine du serveur web (machines virtuelles) est: **192.168.111.134**
* L’adresse IP de la machine KDC (machine virtuelle) est: **192.168.111.133**

Définissons des noms d’hôte pour chaque machine :

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/1.png)

![kdc](/Capture%20d'%C3%A9cran/Kdc/1.png)

![client](Capture%20d'%C3%A9cran/Client/1.png)

Nous pouvons vérifier les adresses IP des trois machines en les exécutant dans chacune d’elles. `hostname -I`

!! Étant donné que le protocole Kerberos implique un horodatage, les horloges des trois machines doivent être synchronisées.

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/0.png)

![kdc](/Capture%20d'%C3%A9cran/Kdc/0.png)

![client](Capture%20d'%C3%A9cran/Client/0.png)

* Machine server web
`hostnamectl --static set-hostname apacheserver.tek-up.de`
![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/1.png)

* Machine KDC
`hostnamectl --static set-hostname kdc.tek-up.de`
![kdc](/Capture%20d'%C3%A9cran/Kdc/1.png)

* Machine cliente
`hostnamectl --static set-hostname client.tek-up.de`
![client](Capture%20d'%C3%A9cran/Client/1.png)

Nous pouvons vérifier le nom d’hôte d’une machine en exécutant la commande: `hostname`

Ensuite, nous allons mapper ces noms d’hôte à leurs adresses IP correspondantes sur les trois machines à l’aide du fichier `/etc/hosts`.
Maintenant, nous devons définir ci-dessous les informations sur /etc/hosts pour les trois machines :
```
<KDC_IP_ADDRESS>    kdc.tek-up.de       kdc
<APACHE_SERVER_ADDRESS>    apacheserver.tek-up.de        apacheserver
<CLIENT_ADDRESS>    client.tek-up.de    client
```

* Machine server web
`$ sudo vim /etc/hosts` ou `$ sudo nano /etc/hosts`
![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/2.png)

* Machine KDC
`$ sudo vim /etc/hosts` ou `$ sudo nano /etc/hosts`
![kdc](/Capture%20d'%C3%A9cran/Kdc/2.png)

* Machine cliente
`$ sudo vim /etc/hosts` ou `$ sudo nano /etc/hosts`
![client](Capture%20d'%C3%A9cran/Client/2.png)

Une fois la configuration terminée, nous pouvons vérifier si les trois machines sont accessibles graçe à la commande `ping`.

Exemple sur la machine cliente:

![client](Capture%20d'%C3%A9cran/Client/6.png)

![client](Capture%20d'%C3%A9cran/Client/7.png)


# Configuration de la machine Centre de Distribution de Clés (KDC):

Voici les packages à installer sur la KDC:
```
   $ sudo apt-get update
   $ sudo apt-get install krb5-kdc krb5-admin-server krb5-config
```
Lors de l’installation, il nous sera demandé de configurer:

   * le royaume: 'TEK-UP.DE' (doit être tout en majuscules)
   ![kdc](Capture%20d'%C3%A9cran/Kdc/3.png)
   
   * le serveur Kerberos: 'kdc.tek-up.de'
   ![kdc](/Capture%20d'%C3%A9cran/Kdc/4.png)
   
   * le server administratif du royaume: 'kdc.tek-up.de'
   ![kdc](/Capture%20d'%C3%A9cran/Kdc/5.png)
   
   * fin d'installation
   ![kdc](/Capture%20d'%C3%A9cran/Kdc/6.png)

**Royaume** ou **Realm** est un réseau logique, similaire à un domaine, auquel appartiennent tous les utilisateurs et serveurs partageant la même base de données Kerberos.

La clé principale de cette base de données KDC doit être définie une fois l’installation terminée :
```
sudo krb5_newrealm
```
![kdc](/Capture%20d'%C3%A9cran/Kdc/7.png)

Les utilisateurs et les services d’un domaine sont définis comme un principal dans Kerberos. Ces principaux sont gérés par un utilisateur admin que nous devons créer manuellement :
```
    $ sudo kadmin.local
    kadmin.local:  add_principal root/admin
```
![kdc](/Capture%20d'%C3%A9cran/Kdc/8.png)

kadmin.local est un programme d’administration de base de données KDC. Nous avons utilisé cet outil pour créer un nouveau principal dans le domaine TEK-UP.DE(). `add_principal`

Nous pouvons vérifier si l’utilisateur "root/admin" a été créé avec succès en exécutant la commande: ``kadmin.local: list_principals``
Nous devrions voir le principal 'root/admin@TEK-UP.DE' répertorié avec d’autres principaux par défaut.

![kdc](/Capture%20d'%C3%A9cran/Kdc/9.png)


![kdc](/Capture%20d'%C3%A9cran/Kdc/10.png)
![kdc](/Capture%20d'%C3%A9cran/Kdc/11.png)
![kdc](/Capture%20d'%C3%A9cran/Kdc/12.png)
![kdc](/Capture%20d'%C3%A9cran/Kdc/13.png)


# Réference et  Inspiration
Ne sachant pas vraiment par ou commencer, je me suis inspirer de:
 * Pour la présentation sur github
    * [Yorsa270](https://github.com/yosra270/postgresql-auth-with-kerberos/)
 
 * Mr Souheib de la chaine Techwall pour l'introduction à kerberos:
    * [Théorique](https://youtu.be/DxlzvDNgkFg)
    * [Pratique](https://youtu.be/vx2vIA2Ym14)

* Installation et configuration et mise en place d'un server web Apache2:
    * [Mettre en place un server web apache](https://youtu.be/arVwa7jvp5M))
