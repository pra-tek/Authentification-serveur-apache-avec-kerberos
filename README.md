# Authentification-serveur-apache-avec-kerberos
Dans ce référentiel, nous avons tentés de mettre en place une authentification du serveur web apache graçe au protocole **Kerberos**.

# Configuration de Kerberos dur Ubuntu
[**Kerberos**](https://fr.scribd.com/presentation/205732355/Kerberos) est un protocole d’authentification qui prend en charge le concept d’authentification unique (SSO). Après s’être authentifiés une fois au début d’une session, les utilisateurs peuvent accéder aux services réseau dans un domaine Kerberos sans s’authentifier à nouveau. Pour que cela fonctionne, il est nécessaire d’utiliser des protocoles réseau compatibles Kerberos.
Dans le cas de HTTP, la prise en charge de Kerberos est généralement fournie à l’aide du mécanisme d’authentification **SPNEGO** (Simple and Protected GSS-API Negotiation). Ceci est également connu sous le nom d'«authentification intégrée » ou « authentification de négociation ». Apache ne supporte pas SPNEGO lui-même, mais le support peut être ajouté au moyen du module d’authentification. `mod_auth_kerb`

![Kerberos en Image](Capture%20d'%C3%A9cran/Kerberos-Protocol.jpg)


# Hôtes et adresse ip

Nous n'avons pas nécéssairement besoin de trois machines pour celà. Une machine peut tout à fait contennir deux rôles ( **KDC** et **server**). Mais dans notre cas, nous avons utilisé trois machines virtuelles sur [VmWare Workstation Pro](https://www.vmware.com/fr/products/workstation-pro/workstation-pro-evaluation.html/). Nos machines étant soit une distribution basé sur ubuntu, soit ubuntu.
Nos machines étant toutes les trois virtuelles, Nous n'avons pas eu besoin de modifier l'adaptateur réseau par defaut (**NAT**) pour les attribuer des addresses ip; Vm Ware s'en est chargé.

Dans notre cas :

* L’adresse IP de l’ordinateur client (machine virtuelle) est: **192.168.111.130**
* L’adresse IP de la machine du serveur web (machines virtuelles) est: **192.168.111.134**
* L’adresse IP de la machine KDC (machine virtuelle) est: **192.168.111.133**

Nous pouvons vérifier les adresses IP des trois machines en les exécutant dans chacune d’elles. `hostname -I`

!! Étant donné que le protocole Kerberos implique un horodatage, les horloges des trois machines doivent être synchronisées.

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/0.png)

![kdc](/Capture%20d'%C3%A9cran/Kdc/0.png)

![client](Capture%20d'%C3%A9cran/Client/0.png)

Définissons des noms d’hôte pour chaque machine :

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


# Configuration des machines:

## Centre de Distribution de Clés (KDC):

Voici les packages à installer sur la KDC:

```
$ sudo apt update
$ sudo apt install krb5-kdc krb5-admin-server krb5-config
```

* Lors de l’installation, il nous sera demandé de configurer:

  * le royaume: 'TEK-UP.DE' (doit être tout en majuscules)

![kdc](Capture%20d'%C3%A9cran/Kdc/3.png)
   
  * le serveur Kerberos: 'kdc.tek-up.de'

![kdc](/Capture%20d'%C3%A9cran/Kdc/4.png)
   
  * le serveur administratif du royaume: 'kdc.tek-up.de'

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

[**kadmin.local**](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/admin_commands/kadmin_local.html) est un programme d’administration de base de données KDC. Nous avons utilisé cet outil pour créer un nouveau principal dans le domaine TEK-UP.DE(). `add_princ`

Nous pouvons vérifier si l’utilisateur "root/admin" a été créé avec succès en exécutant la commande: ``kadmin.local: list_principals``

Nous devrions voir le principal 'root/admin@TEK-UP.DE' répertorié avec d’autres principaux par défaut.

![kdc](/Capture%20d'%C3%A9cran/Kdc/9.png)

Ensuite, nous devons accorder tous les droits d’accès à la base de données Kerberos à admin principal root / admin en utilisant le fichier de configuration /etc/krb5kdc/kadm5.acl .

``sudo vim /etc/krb5kdc/kadm5.acl``

Dans ce fichier, nous devons ajouter la ligne suivante :

```
*/admin@TEK-UP.DE  *
```

![kdc](/Capture%20d'%C3%A9cran/Kdc/10.png)

Pour que les modifications prennent effet, nous devons redémarrer le service suivant:

```
sudo service krb5-admin-server restart
```

Une fois que l'utilisateur "admin" qui gère les principaux est créé, nous devons créer les principaux. Nous allons crééer des principaux pour la machine cliente et la machine serveur de web.

* **Créons un mandataire pour le client** :

```
 $ sudo kadmin.local
kadmin.local:  add_principal jean
```

![kdc](/Capture%20d'%C3%A9cran/Kdc/11.png)
   
* **Créons un principal pour le serveur de service** :

```
kadmin.local:  add_princ -randkey apacheserver.tek-up.de
```

![kdc](/Capture%20d'%C3%A9cran/Kdc/12.png)

Nous pouvons vérifier la liste des principaux en exÃ©cutant la commande:

```
kadmin.local: list_principals
```

![kdc](/Capture%20d'%C3%A9cran/Kdc/13.png)

## Le serveur web apache:

Installation de Apache2 :

```
 $ sudo apt update -y
 $ sudo apt install apache2 -y
```

Suite à l'installation, le serveur Apache est déjà démarré, on peut le vérifier avec la commande ci-dessous. Cela permettra de voir qu'il est bien actif.

```
$ sudo systemctl status apache2
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/3.png)
      
Pour que notre serveur demarre automatiquement au démarrage de la machine, on doit executer la commande:

```
$ sudo systemctl enable Apache2
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/4.png)
      
Après l'installation, il est recommandé d'installer "curl" avec la commande ci-dessous

```
$ sudo apt install curl
```

Les fichiers et dossiers necessaires à la configuration  des sites webs au travers des hôtes virtuels sont dans ``/etc/apache2``. ceux nous intéressant sont:
	
* Le fichier ``apache2.conf``:
      
> Contenant la configurations par defauts d'apache.
   
* Le dossier ``conf-available``:
      
> Contenant les configurations disponibles dans apache.
   
* Le dossier ``conf-enabled``:
      
> Contenant les configurations actives dans apache.
   
* Le dossier  ``mods-available``:
      
> Contenant les modules prient en charges par apache.
   
* Le dossier ``sites-available``:
      
> Contenant les fichiers de configuration des sites web.
      
* **NB**:
> Apache lit les fichier de  configuration pas ordre numérique de 000 à XXX.

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/9.png)
   
* Le dossier ``sites-enabled``:
      
> Contenant les fichiers  des sites actif sur le serveur.
      
![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/5.png)
   
### Configuration du serveur pour  notre site

Pour ce Projet nous avons décider de changer la page html par défaut du serveur par un site web basic (sans Js  ni Php).

* Nous allons créer un site sur notre serveur. Pour ma part, ce sera le site **Cyberias.git**, accessible également sur [cyberias](https://pra-tek.github.io/cyberias/). Il sera stocké à l'emplacement suivant : /var/www/cyberias.

```
$ sudo mkdir /var/www/cyberias
```

* ``www-data`` etant l'utilisateur d'apache appartenant au groupe ``www-data``, nous allons changer le propriétaire de notre dossier ainsi que son groupe. Et vérifier l'effectiviter des changement graçe à ``ls``:

```
$ sudo chown -R www-data:www-data /var/www/cyberias/
$ ls /var/www/cyberias -la
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/7.png)

* Créons le fichier de configuration de notre site:

```
$ sudo vim sites-available/001-cyberias.conf
ou bien
$ sudo vim /etc/apache2/sites-available/001-cyberias.conf
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/19.png)

  * Configurons:

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/15.png)

* Apache  possède un outils  de verification des fichiers de configurations nommé ``configtest``, qui éffectu un test de ces fichiers (la syntaxe, indentation, etc ...). Il est accéssible par la commande:

```
$ /usr/sbin/apachectl configtest
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/10.png)

* Nous devons activer notre site web. En créant un lien symbolique ``sites-available`` vers ``sites-enabled`` graçe à la commande:

```
$ sudo a2ensite 001-cyberias
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/20.png)
	
* Nous avons créer un fichier ``.htaccess`` pour la gérer la réecriture de notre url. afin que chaque fois que nous tapons ``www.cyberias.com`` l'url est reécrit en ``cyberias.git``

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/11.png)
         
* Pour éviter les érreurs ``FORBIDEN`` à cause de l'absance du fichier ou plutôt de lien symbolique de **rewrite** dans ``mods-enabled``, nous avons activer le module rewrite.

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/12.png)
	    
![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/13.png)
            
![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/14.png)
      
* Nous avons modifier le fichier de configuration de la page par defaut afin qu'elle pointe ver notre site.

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/16.png)
         
  * Vérifions l'effectiviter de notre config dans ``sites-enabled/``:
	
![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/22.png)
      
  * pour que nos modification soit prise en compte, nous devons redemarrer notre serveur:

```
$ sudo systemctl reload apache2
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/21.png)
           
### Configuration Kerberos

* Installation des Packages nécessaires:
  
  * le package ``libapache2-mod-auth-kerb`` 

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/23.png)
  
  * le package ``krb5-user``

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/29.png)

* Configuration de l'installation:
  * Le royaume:

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/24.png)
  
  * Le serveur kerberos:

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/25.png)
  
  * Le serveur administrateur du royaume:

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/26.png)

#### Préparation du fichier keytab

Nous devons extraire le principal du service de la base de données des principaux KDC  dans un fichier keytab.

* Dans la machine KDC, exécutons la commande suivantes pour générer le fichier keytab:

```
$ sudo kadmin.local
kadmin.local: ktadd HTTP/apacheserver.tek-up.de@TEK-UP.DE
kadmin.local: q
```

![kdc](/Capture%20d'%C3%A9cran/Kdc/14.png)

* Vérifions que notre keytab a été créer, graçe à l'utilitaire ``klist``:

```
sudo klist -kt /etc/krb5.keytab
```

![kdc](/Capture%20d'%C3%A9cran/Kdc/15.png)

* Envoyez le fichier keytab de la machine KDC à la machine du serveur:
! Nous devons avoir openssh-server package installé sur le serveur:
``sudo apt install openssh-server``
![kdc](/Capture%20d'%C3%A9cran/Kdc/16.png)

* Vérifiez que le principal du service a été extrait avec succès de la base de données KDC:

  * Répertorier la liste de clés actuelle:
``ktutil:  list``

  * Lire un keytab krb5 dans la liste de touches actuelle
``ktutil:  read_kt /home/orphe/Bureau/krb5.keytab``
      
  * Répertorier à nouveau la liste de clés actuelle
``ktutil:  list``

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/30.png)

#### Configuration de notre site :

* Modifions légèrement  la configuration de notre site dans le fichier ``001-cyberias.conf``, afin d'intégrer l'authentification kerberos. Ajouter ce qui suit dans ``<VirtualHost *:80> ...... </VirtualHost>`` .

```
<Location /
	AuthType Kerberos
	AuthName "ce que vous voulez"
	KrbAuthRealms TEK-UP.DE
	KrbServiceName HTTP/apacheserver.tek-up.de
	KrbMethodNegotiate on
	KrbMethodK5Passwd off
	Krb5Keytab /home/orphe/Bureau/krb5.keytab
</Location>
```

![apacheserver](Capture%20d'%C3%A9cran/Apacheserver/33.png)

**NB**:
> La ``require valid user`` doit rester commenter ou ne même pas exister, sauf si vous voulez definir une liste d'utilisateur spécifique.


## Le client

Notre serveur est bien accessible dépuis la machine cliente:
![client](Capture%20d'%C3%A9cran/Client/3.png)

![client](Capture%20d'%C3%A9cran/Client/3--1.png)

### Préparation de Kerberos

* Installation de ``krb5-user``:

![client](Capture%20d'%C3%A9cran/Client/9.png)

  * Le royaume:

![client](Capture%20d'%C3%A9cran/Client/10.png)
  
  * Le serveur kerberos:

![client](Capture%20d'%C3%A9cran/Client/11.png)
  
  * le serveur administratif:

![client](Capture%20d'%C3%A9cran/Client/12.png)

### Authentification du client

* Dans la machine cliente, vérifiez les informations d’identification mises en cache :
``$ klist``

* Initialez ensuite l’authentification de l’utilisateur :
``$ kinit jean@TEK-UP.DE``

![client](Capture%20d'%C3%A9cran/Client/15.png)

* Et vérifiez le ticket d’octroi de ticket (TGT) :
``$ klist``

> Si elle ne contient pas une liste d'utilisateur et n'est plus commenter, le client ne poura pas accerder au serveur.

![client](Capture%20d'%C3%A9cran/Client/16.png)

![client](Capture%20d'%C3%A9cran/Client/17.png)

![client](Capture%20d'%C3%A9cran/Client/18.png)

**La configuration est parfaite** le client à accès au serveur

![client](Capture%20d'%C3%A9cran/Client/19.png)

# Réference de travail

 * Pour la présentation sur github
    * [Yorsa270](https://github.com/yosra270/postgresql-auth-with-kerberos/)
 
 * Mr Souheib de la chaine **Techwall** pour l'introduction à kerberos:
    * [Théorique](https://youtu.be/DxlzvDNgkFg)
    * [Pratique](https://youtu.be/vx2vIA2Ym14)

* La chaine **Grafikart** pour l'installation, la configuration et mise en place d'un server web Apache2:
    * [Mettre en place un server web apache](https://youtu.be/arVwa7jvp5M))
