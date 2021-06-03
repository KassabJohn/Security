DOSSIER TECHNIQUE

Durée :

    32h

Introduction :

    Sécurité des systèmes d’informations (Linux/Windows)
    Établir les bases nécessaires à la compréhension de la problématique (concepts, terminologie, classification)
    Dresser un panorama global des risques et des solutions
    Illustrer ces concepts et solutions par des réalisations pratiques
    Les positionner dans le contexte du monde actuel via des études de cas
    Comprendre ce qu'est la PRA (Plan de Reprise d'Activité) et la redondance pour la continuité des services

Solutions à mettre en œuvre :

    Le firewall devra remplir les rôles de : Gestion des flux, Proxy, Reverse Proxy, Détection d'intrusion
    Le contrôleur de domaine devra permettre la gestion du DNS local et des GPO qui seront appliquées au poste client
    La gestion des flux entrants et sortants devra être gérée par le firewall, seuls les ports 80 et 443 seront autorisés
    Le Proxy lié au service SquidGuard, devra permettre la gestion du blocage de noms de domaines / url etc en sortie
    Le Reverse Proxy devra permettre l'accès au travers d'un nom de domaine unique et de ses 2 sous-domaines à 2 Virtual Host (ports 80 et 81) hébergés sur le serveur Web, le tout devra être chiffré sur le protocole https côté WAN du firewall
    La configuration du Proxy sera poussée sur les navigateurs Edge et Firefox des postes clients par GPO

Mise en place d\'une infrastructure comprenant :

-   1 firewall

-   1 serveur Web

-   1 contrôleur de domaine

-   1 client

Année 2020 - 2021

L'objectif de ce dossier technique est de mettre en place une
architecture similaire à celle-là :

![](mediafolder/b29bc4867c2e3658a3329ff3fbe8789253c86548.png)

Installation + configuration de l'ISO pfSense

Create a new VM → Custom → workstation 16.x

I will install the operatin system later pour empecher la creation de
fichiers de reponses automatiques

FreeBSD 12-64 bits → version

Nb process → 1

Nb core → 2 / 4

Memory of this virtual machine → 512 MB

Network type → NAT

Disk → 40G → store virtual disk as a single file

On ajoute 2 nouvelles cartes réseaux secu-DMZ / secu-LAN

On fait une snapshot

On ajoute l'iso pfsense au CV/DVD puis on exec

On accepte, choix langue, choix BIOS puis on installe

On config l'IP du WAN sur 192.168.5.54 sur la passerelle 192.168

On configure la 3^e^ carte réseau non reconnu (em2) en tant que LAN

![](mediafolder/2c118d56bc9461a91227ba7c4e2cc0dba65dd562.png)
On définit le LAN sur l'adresse 192.168.15.54

On démarre une VM Client que l'on met sur le segment secu-LAN

On modifie l'adresse IP :

![](mediafolder/7996f30def7c8f4a00a5e1675a7db90280c48d27.png)

Voulez-vous autoriser les autres pc de ce réseau a détecter votre pc →
oui

Pour accéder à l'interface Web de pfSense on se rend dans son navigateur
à l'adresse 192.168.1.54

Les identifiants par défaut de pfSense : id : admin / mdp : pfsense

![](mediafolder/5645ee73d6643938fb3e16bf3cda7bed795195d9.png)

![](mediafolder/635ca5e180365a6be724fa2f8f2a2176e61cb530.png)

![](mediafolder/a26b4f69ce97c356f3f749d57076c88460c1c56f.png)

![](mediafolder/461fe1f8e441c34dddd59eb5c421c07b54268ffe.png)

![](mediafolder/ba22a259da4e8f6c4ba3e84a27bb9095bf39791a.png)

On va dans firewall → Rules → OPT1(em2)

On active l'interface (enable)

![](mediafolder/0ac7d2aa98affda6672375149b5c02176136f71e.png)

On se retrouve avec nos 3 cartes

![](mediafolder/5a39d24847acee858b09af18839505dbcfd5aab0.png)

Voici la configuration rules de LAN pour l'instant

![](mediafolder/65919a6b86e65f9c01aca0c5a4e4181fe6aa4462.png)

![](mediafolder/dcf71cf7780c8f5a32e43dc82a98eba96e37b875.png)

On désactive les règles par défaut sur l'interface LAN

![](mediafolder/cc0278fc4d81cba75d4256118d9843a29d3f8820.png)

Cependant on laisse activer la Rule Anti-lockout Rule car elle autorise
l'adresse ip du firewall l'accès au port 443 et 80. Elle autorise
n'importe quel client sur l'interface LAN a accéder à l'interface web à
partir des ports 443 et 80.

On crée plusieurs règles qui permettront la sortie de la zone LAN vers :
la résolution DNS, le PING, http, https vers \"tout\"

Les règles sont lues de haut en bas, on applique donc la règle DNS en
haut de la liste

On ajoute donc 4 Rules en cliquant sur le bouton « add »

-   Le protocole ICMP (test : ping 8.8.8.8 en cmd)

-   Le port 53 pour la résolution de noms (DNS)

-   Le port 80 et 443 (http/https)

Règle qui permet la sortie LAN vers la résolution DNS :

![](mediafolder/5d2d20210257b87e2b64d50ac784f274b143d1b4.png)

Règle qui permet la sortie LAN vers le PING

![](mediafolder/c907c25af885fcefccbf5d3f749eff9139ff9c9e.png)

Règle qui permet la sortie LAN vers le HTTP & l'HTTPS

![](mediafolder/2f3c2eaa4557b563d30d8d7a7a2f3db65738cec3.png)

![](mediafolder/89e24bf5fe1ed24158385acde7b8ec1143143ade.png)

On télécharge une iso Ubuntu mini

On la connecte au segment LAN-DMZ qui correspond à notre réseau pour le
serveur web

On configure le fichier 01.netcfg.yaml dans etc/netplan en désactivant
le DHCP (pas de DHCP pour les serveurs). On lui attribue une adresse IP,
une gateway et un DNS

![](mediafolder/38d947ffb42a5181a13ab51dbf01c31b84d9ad68.png)

Pour autoriser l'accès au port 81 du serveur Web, on doit spécifier en
plus l'écoute au port 81 directement dans le fichier port.conf d'Apache

![](mediafolder/0e326bce6070976b752a561cf9c5ea98341f2cd6.png)

On applique les changements avec systemctl restart apache2

Pour pouvoir accéder à Internet, il faudra dans pfSense ajouter une
nouvelle règle :

Firewall → Rules → OPT1 : protocol : any ; from any to any

Dans le serveur web on applique les changements : sudo netplan apply

On met tout à jour avec apt-get update

Nous allons désormais installer Apache2 pour mettre en place notre
serveur

On exécute apt install apache2

On configure le fichier de conf dans /etc/apache2/sites-available

On y ajoute le port et le ServerName.

![](mediafolder/b4eb6e3b4a7913ba1cf59eba5fe403de19907b0d.png)

On fait de même pour site1.contoso.web sur le port 81

On interdit l'accès à la zone DMZ vers la zone LAN grâce à la règle
suivante que l'on applique à partir du LAN vers OPT1 et on autorise les
requêtes du port 80,53, 443 ainsi que les requête ICMP

![](mediafolder/98d21bcc731c8f31d3adfcbd50dd88976741fb6c.png)

![](mediafolder/cfa4902bc5afb74b3b1dabb4e3a5e7d4f3893f18.png)

On ajoute une règle qui permettra l'accès depuis la zone LAN aux ports
80 et 81 du serveur Web

Pour le port 80 :

![](mediafolder/391b0c28059f29f87ae91cf6354f792a75aa2d9a.png)

Et pour le port 81 :

![](mediafolder/1b3a3b6521023e08f242a8218960a98a7cf54c6b.png)

![](mediafolder/1b3a3b6521023e08f242a8218960a98a7cf54c6b.png)

Nous allons désormais installer et configurer le service SquidGuard qui
va nous permettre la gestion du blocage de noms de domaine et URL

Il faudra tout d'abord installer les packages « squid » et
« squidGuard »

On se rend dans System → Package Manager → Availables Packages

![](mediafolder/8477bee8424061d74e481f53d37d672934440226.png)

On accède ensuite au SquidProxyServer dans l'onglet général pour
l'activer.

![](mediafolder/d96b63545a388fa9b6aacfa2e7c4b946cf7250a1.png)

On active le SquidGuard Proxy Filter pour pouvoir filtrer certains URL.

![](mediafolder/3de9554b3f9cbc047f766ec67949749cf0055f6b.png)

Pour gérer le blocage d'URL à partir d'une blacklist, on se rend dans
« Services → SquiGuard Proxy Filter → Blacklist »

Activer « Enable Blacklist » et insérer dans BlackList URL:
<http://dsi.ut-capitole.fr/blacklists/download/blacklists_for_pfsense.tar.gz>

![](mediafolder/b8b0d5fbb08d509117f172c43c11bf95ea335725.png)

Dans l'onglet « Blacklist », on clique sur « Download » pour télécharger
la liste de filtrage.

![](mediafolder/512ec8c797ecd57855791aca4980b62f36ff411d.png)

Dans l'onglet « Common ACL », on clique sur « Target Rules List » sur le
« + » puis on sélectionne les catégories à bloquer et on sélectionne
« allow » pour « default access \[all\] »

![](mediafolder/eef290be9f443aab547a4e7cfb4b7978f34e72bf.jpg)

Nous allons devoir créer une GPO pour configurer le proxy de Mozilla
Firefox et Microsoft Edge

Pour Microsoft Edge on se rend dans les options de proxy du navigateur :

![](mediafolder/5b2f22a6c97753c553bf474c6a704410731558cc.png)

Puis nous allons télécharger et installer des stratégies liées au
navigateur Firefox :

Pour cela, nous nous rendons sur le site :
<https://github.com/mozilla/policy-templates/releases> et télécharger le
fichier Policy_templates_v2.10.zip

Ensuite nous allons dans C:\\Windows\\ et nous copions le dossier
PolicyDefinitions et nous le collons dans
D:\\ADDS\\SYSVOL\\domain\\Policies\\PolicyDefinitions

Puis nous copions les dossiers fr-FR, en-US, les fichiers mozilla.admx
et firefox.admx de l'archive dans le dossier PolicyDefinition

![](mediafolder/a24cfbc9dedb2a3e16d5cccfb2a5126b1e153f65.png)

Dans les paramètres de notre stratégie de groupe, le dossier Mozilla
apparait, on peut alors activer les paramètres de proxy :

![](mediafolder/74612c481285b0dfe9cc6d82c34e7bf7d42d7799.png)

Nous allons ensuite créer une GPO qui va gérer le \"push\" du certificat
racine de pfSense sur les postes clients

![](mediafolder/ca946af522588a1853d07d3e3385cbf6e0d40d42.png)

**Question légalité : est-il acceptable d\'intercepter le trafic https ?
Quel est le nom et le fonctionnement de cette méthode ?**

Il est possible de contrôler le contenu des données échangées entre le
client et le serveur afin de s'assurer que les flux HTTPS ne sont pas
utilisés pour faire sortir du système d'information des données
confidentielles. Cependant le contrôle de données n'est pas toujours
utilisé de cette manière. Certaines personnes utilisent ce
fonctionnement pour récupérer des informations personnelles etc.

Le fonctionnement de cette méthode se nomme : man-in-the-middle.

Nous allons désormais configurer l'utilisation de Squid sur les postes
clients avec WPAD.

Un serveur Web peut être monté sur la zone LAN, il fonctionnera sur le
port 80 et hébergera à sa racine un fichier « wpad.dat ».

Tout d'abord il faut créer un serveur web avec UWAMP qui hébergera ce
fichier puis démarrer le serveur

![](mediafolder/08963c09434324ac5a4529c6a4b8d6f45ba2b34a.png)

Nous allons maintenant créer une règle DNS pour servir le fichier à
l'adresse : wpad.contoso.adds

On crée donc un hôte dans la zone de recherche directe contoso.adds,
avec l'extension wpad pointant sur notre serveur.

![](mediafolder/3debfee662b15f0b93d77154eaef7a8c5e453c20.png)

Il faut désormais vider le regedit. On lance l'invite de commande en
tant qu'administrateur et on exécute la commande suivante :

-   Dnscmd /info /globalqueryblocklist

Puis on supprime les valeurs du regedit à l'emplacement :

HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\DNS\\Parameters/GlobalQueryBlockList

![](mediafolder/278180e31e4db392fb62c2b3fff0709d79d9898a.png)

Pour vider le cache, on lance l'invite de commande en administrateur et
on exécute la commande suivante :

-   ipconfig /flushdns

Nous allons désormais créer un fichier wpad.dat dans lequel nous allons
définir et configurer le proxy.

Ce fichier sera servi par le serveur Uwamp et permettra de configurer
les navigateurs clients

![](mediafolder/cf1800f45ac75749a7fdb2813d82ef56c7be4141.png)

Nous allons maintenant configurer le Reverse Proxy.

Pour se faire, nous devons rediriger les sous domaines site0.contoso.web
et site1.contoso.web vers le serveur web respectivement sur les ports 80
et 81.

Nous allons activer Squid Reverse Proxy sur l'interface WAN.

Pour ajouter nos 2 sites, il faut avant tout les ajouter dans « Web
Servers » en précisant l'IP de destination de notre serveur web à
l'adresse IP : 192.168.10.11 et les ports affectés : 80 puis 81

![](mediafolder/d4cfcaced88d72ec0e4035c5b1853edb2c20dff3.png)

Nous allons également utiliser des alias pour ajouter nos 2 serveurs
dans des groupes et leur spécifier une URL dans Mappings afin qu'il
continue de réaliser son rôle.

![](mediafolder/7ca55b22058a623c2b5825d65b02ce4609a32e7c.png)

Nous devons aussi rediriger les requêtes que l'on va recevoir.

Nous modifions donc le fichiers hosts dans /etc/hosts :

![](mediafolder/bd85cdaa6d576e3760961b19d12e9d9342e85949.png)

Il faut maintenant activer le « reverse proxy http » sur le port 80.
Pour cela il faut modifier le port minimum qui est demandé pour le
reverse proxy. Il faut donc ajouter une nouvelle « Tunnables » nommé
**« **net.inet.ip.portrange.reservedhigh** »** et mettre la valeur à 0 

![](mediafolder/f1f5d71a826016bb32d3efa4e47cb859c34ae90d.png)

Pour tester le Reverse Proxy, nous allons accéder a
« C:\\Windows\\System32\\drivers\\etc ».

Nous allons déplacer le fichier hosts en dehors de son dossier car nous
n'avons pas les droits d'édition, et nous allons rajouter à l'intérieur
l'ip de notre serveur web. Il faudra ensuite le remplacer par le fichier
original présent dans le dossier.

On peut désormais ping [www.contoso.web](http://www.contoso.web) sans
problème et accéder à nos sites web sans problème.

![](mediafolder/a3056a5d26a2ff969004e1cc2b53889a0cf32e60.png)

**Problèmes rencontrés :**

Il est primordial de vider le cache avec la commande « ipconfig
/flushdns » après avoir supprimé les valeurs du regedit afin que les
changements soient pris en compte.
