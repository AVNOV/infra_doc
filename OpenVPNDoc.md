# OpenVPN

Source : "https://www.it-connect.fr/pfsense-configurer-un-vpn-ssl-client-to-site-avec-openvpn/"

Ce document d'installation est rédigé pour permettre la configuration d'OpenVPN sur pfsense.

Le but est de configurer un VPN SSL client-to-site sous PfSense via OpenVPN pour permettre à vos PCs d'accéder à distance aux ressources de l'entreprise.

## I Les certificats

Nous devons créer une autorité de certification interne sur le firewall PfSense, puis nous allons créer un certificat dédié au serveur. Ce certificat sera utilisé pour sécuriser notre tunnel VPN.

### 1 Créer l'autorité de certification

Aller sur System puis Cert.Manager.

Dans l'onglet "CAs", cliquez sur le bouton "Add".

Donnez un nom à l'autorité de certification, par exemple "CA-ITCONNECT-OPENVPN", ce nom sera visible seulement dans Pfsense. 

Choisissez la méthode "Create an internal Certificate Authority".

Le champ "Common Name" est celui qui sera affiché dans les certificats.

Remplissez les autres champs à votre convenance et cliquez sur "Save"

A partir de la, votre autorité de certification nouvellement créée devrait apparaître dans System/Certificate Manager/CAs

### 2 Créer le certificat Server

Nous devons créer un certificat de type "Server" en nous basant sur notre nouvelle autorité de certification. Toujours dans "Certificate Manager", cette fois-ci dans l'onglet "Certificates", cliquez sur le bouton "Add/Sign".

Choisissez la méthode "Create an Internal Certificate" puisqu'il s'agit d'une création, donnez-lui un nom (VPN-SSL-REMOTE-ACCESS) et sélectionnez l'autorité de certification au niveau du paramètre "Certificate authority".

Par défault, la validité du certificat sera fixé à 10 ans. Il est préférable, dans le champ Common Name, d'indiquer votre nom de domaine.

Attention, assurez-vous que le type de certificat soit bien Server Certificate.

Après avoir cliqué sur "Save" pour valider la création du certificat, il apparaît dans la liste des certificats du Pare-feu sur System/Certificate Manager/Certificates.


## II Créer les utilisateurs locaux

Maintenant que le certificat server est créé, il nous faut un certificat utilisateur. 

Pour créer l'utilisateur, il faut indiquer un identifiant, un mot de passe... Ainsi que cocher l'option "Click to create a user certificate" : cela va ajouter le formulaire de création du certificat juste en dessous. Pour créer le certificat, on se base sur notre autorité de certification (dans le champ certificate authority).

Une fois cet utilisateur créé, il apparait dans System/User Manager/Users

## III Configurer le serveur OpenVPN


### Config partie 1 :

Cliquez sur le menu VPN, puis sur OpenVPN.

Dans l'onglet Servers, cliquez sur Add pour créer une nouvelle configuration.

La première chose à faire, c'est de choisir le "Server Mode" suivant : Remote Access (SSL/TLS + User Auth).

Pour le VPN, le protocole fonctionne sur de l'UDP. Le port 1194 est le port utilisé par défaut, ce qui veut dire que tout le monde le connait et qu'il est donc conseillé d'en changer.

L'interface à séléctionner est bien l'interface WAN, car c'est par celle-ci que nous allons recevoir le connexions clients.

Au niveau du champ "Peer Certificate Authority", séléctionnez votre autorité de certification créée précédemment.
Puis au niveau du champ "Server certificate", séléctionnez le certificat Server que nous avons généré.

Séléctionnez ensuite l'algorythme de chiffrement que vous souhaitez utiliser. 

### Config partie tunnel :

Le champ "IPv4 Tunnel Network" correspond à l'adresse réseau VPN. Un client qui se connecte au VPN recevra une adresse IP dans ce réseau. 

Le champ "Redirect IPv4 Gateway" coché signifie que l'entièreté des flux réseau du PC distant vont passer dans le VPN.

Le champ "IPv4 Local network" est à remplir avec les adresses réseaux des LAN (séparés par une virgule) que l'on souhaite rendre accessible via notre tunnel VPN. 

Le champ "Concurrent connections" correspond au nombre de connexions VPN simultanés que vous autorisez.

Au niveau de la "Topology", remarque très importante à prendre en compte : pour des raisons de sécurité, il vaut mieux utiliser la topologie "net30 - isolated /30 network per client" pour que chaque client soit isolé dans un sous-réseau (de la plage réseau VPN) afin que les clients ne puissent pas communiquer entre eux.

Pour la configuration DNS, il est possible d'utiliser la résolution interne de l'entreprise en configurant la partie DNS Server 1 et en cochant la case "Provide a DNS server list to clients. Addresses may be IPv4 or IPv6" (vous pouvez aussi indiquer un nom de domaine par défaut).

Dans la zone "Custom options", indiquez : auth-nocache. Cette option offre une protection supplémentaire contre le vol des identifiants en refusant la mise en cache.

Vous pouvez enfin valider la configuration.

## IV Exporter la configuration OpenVPN

Pour télécharger la configuration au format ".ovpn", il est nécessaire d'installer un paquet supplémentaire sur notre pare-feu. Rendez-vous dans le menu suivant : System > Package Manager > Available Packages.

Recherchez "openvpn" et installez le paquet : openvpn-client-export.

Si vous souhaitez utiliser l'adresse IP publique pour vous connecter, utilisez l'option "Interface IP Address" pour l'option "Host Name Resolution". Il y a d'autres options possibles, notamment par nom de domaine.

Les autres options peuvent être laissées par défaut... Il y a seulement notre option "auth-nocache" a reporter dans la section des options additionnelles.

Cliquez ensuite sur le bouton "Save as default".

En dessous de la part configuration, vous avez la possibilité de télécharger la configuration. 

Pour utiliser OpenVPN Community (ce qui nous interesse ici), il faudra prendre la configuration "Bundled Configuration", au format archive pour récupérer tous les fichiers nécessaires.

## V Créer les règles de firewall pour OpenVPN

D'une part, nous devons créer une règle pour autoriser les clients à monter la connexion VPN, et d'autre part nous devons créer une ou plusieurs règles pour autoriser l'accès aux ressources : serveur en RDP, serveur de fichiers, application web, etc.

### Autoriser le flux OpenVPN

Cliquez sur le menu "Firewall" > "WAN". Il est nécessaire de créer une nouvelle règle pour l'interface WAN, en sélectionnant le protocole UDP.

La destination ce sera notre adresse IP publique donc sélectionnez "WAN address". Pour le port, prenez OpenVPN dans la liste ou alors indiquez votre port personnalisé.

Si vous le souhaitez, vous pouvez ajouter une description à cette règle et activer les logs.

Validez la création de la règle et appliquez la configuration.

À partir de ce moment-là, il est possible de monter le tunnel VPN sur un PC, mais les ressources de votre entreprise seront inaccessibles.

### Autoriser les flux vers les ressources

Ajoutez une nouvelle règle, cette fois-ci sur l'interface OpenVPN. 

La règle qui suit sert à autoriser l'accès en RDP à l'hôte (qui fait bien parti du réseau autorisé dans la configuration du VPN) au travers du tunnel VPN. Vous devez créer une ou plusieurs règles en fonction des ressources auxquelles vos utilisateurs doivent accéder via le VPN, en limitant les flux au maximum.

Pour la destination, ce sera donc mon hôte et le port 3389 pour le RDP. De la même façon que pour la règle précédente, indiquez une description et activez la journalisation si vous le souhaitez.

Une fois cela fait et la règle validé, la configuration du VPN est terminée.

## VI Tester l'accès distant depuis un poste client

En premier lieu, téléchargez et installez le client OpenVPN sur le poste client.

Sur windows : 

Dans le dossier "C:\Programmes\OpenVPN\Config" vous devez extraire le contenu de l'archive ZIP téléchargée depuis le Pfsense et qui contient la configuration. Vous pouvez créer un sous-dossier dans le dossier "config" si vous voulez.

Ensuite, sur l'icône OpenVPN effectuez un clic droit et cliquez sur "Connecter".

Vous devez fournir le nom d'utilisateur et le mot de passe, correspondant au compte local du pare-feu.

Lorsque le tunnel VPN est monté et actif, l'icône devient vert.

Si l'on effectue un ipconfig sur le PC, nous pouvons voir que l'on a bien une adresse IP sur la plage 10.10.10.0, avec un sous-réseau en /30 pour l'isolation des clients.
Il ne reste plus qu'à établir une connexion sur vos serveurs, via RDP, Web, ou autre, selon vos besoins 

L'accès VPN SSL pour vos utilisateurs nomades ou en télétravail est désormais opérationnel ! 