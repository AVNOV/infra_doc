# NetData

Ce document d'installation est rédigé pour une installation sur centos7.

## Pré-requis :

-   une machine centos7 fonctionnelle avec un acces internet 
-   les packages de la machine sont à jour ("sudo yum check-update" pour vérifier, "sudo yum update" puis reboot de la machine pour le faire).
-   Vous êtes connecté à la machine avec au minimum des privilèges sudo


## Installation :

En premier lieu, il faut installer les dépendances dont netdata a besoin : 

`
sudo yum install zlib-devel libuuid-devel libmnl-devel gcc make git autoconf autogen automake pkgconfig
`

`
sudo yum install curl jq nodejs
`

Ensuite, l'installation de netdata s'effectue grâce à l'execution d'un script bash :

`
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
`

## Acceder à l'interface web de netdata :

pour pouvoir avoir acces à l'interface web de nedata, il vous faut d'abord ouvrir le port correspondant sur votre firewall :

`
sudo firewall-cmd --permanent --zone=public --add-port=19999/tcp
`

`
sudo firewall-cmd --reload
`

L'interface web de monitoring est maintenant accessible à l'url http://localhost:199999.

## Voir plus loin avec Prometheus et Grafana

netdata permet de monitorer efficacement un serveur en local. Mais pour pouvoir monitorer des machines depuis une machine administratrice, il est nécessaire d'installer la suite Prometheus et Grafana. 

Prometheus va servir à récolter et stocker les métrics des serveurs et grafana servira quant à lui a organiser et afficher ces différentes données récoltées.

Si vous souhaitez déployer ces deux services, installez docker et utilisez le fichier docker-compose_exemple.yml avec la commande

`
docker-compose up -d
`

Afin que Prometheus cible les métics fournies par netdata, il vous faudra aussi modifier le fichier de configuration prometheus.yml pour indiquer les adresses des machines à monitorer. 

Dans la partie scrape_configs du fichier, trouver la ligne "targets" et l'éditer comme ceci (en remplaçant par l'ip de votre serveur à surveiller) :


`scrape_configs:`

`- job_name: 'netdata'`

`metrics_path: '/api/v1/allmetrics?format=prometheus'`

`scheme: 'http'`

`static_configs:`

`- targets: ['%%IP_SERVER%%:19999']`


