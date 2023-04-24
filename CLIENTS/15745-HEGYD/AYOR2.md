---
title: AYOR
description: Création VM / demande 20 avril 2023
published: 1
date: 2023-04-24T10:48:30.309Z
tags: ayor
editor: markdown
dateCreated: 2023-04-24T10:24:32.850Z
---

# Projet Ayor


## Description

Serveur : ayor.hegyd.net
4vCPU 3,4 GHZ, 6Go RAM , 250 GB de stockage
Debian 11.3
PHP-FPM 8.1.5

**Utilisateurs**

En plus des développeurs, il faut ajouter l'utilisateur "integration_continue" avec la clef SSH :

	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCnYVCWi4Fr+xnT664/MRGxH8q7IggNwsmUHmCqYhcMVoTW7vZ18CdZ6Zo5YLpV8HAsSNHpkGMQexhe5JTo9ZYXFaG5cOlsiIUDTwHO6oaiGRVnOl5uhj2cqJtgfuoqmw+a8xtEv79gA3K8NZ3xxTAW8S/yN6C4uMQkq3aBx7MDvmva72FsogJ5e9dtXJy9CMgpcDB6YUtNQjm0dzBqky42p43VnbpvDR0EqUvbvFPo62TnepLv5C9CvUSDOlo4Wu2kSCZF0xzFtyo5PN0GO+Z/FVYQxxhkJfx+H+aCYcia04r0wr4cEvdxtjAu0LlBRus7MY/l0xWplz+b1PkDiib3 your_email@example.com`

Cet utilisateur doit avoir les mêmes droits que les développeurs.
De plus il faut autoriser l'IP suivante dans le firewall du serveur : 51.255.99.74 (VM sur proxmox).

**Création des environnements du projet**

"ayor" étant le nom du projet au sein du serveur.

- recette

    DocumentRoot : /var/projects/ayor/recette/current/public
    URL : recette.ayor.hegyd.net
    Base de données : ayor_recette

- preprod

    DocumentRoot : /var/projects/ayor/preprod/current/public
    URL : preprod.ayor.hegyd.net
    Base de données : ayor_preprod

- prod

    DocumentRoot : /var/projects/ayor/prod/current/public
    URL : prod.ayor.hegyd.net (en attendant le domaine définitif)
    Base de données : ayor_prod


**SSL**

Mettre en place des redirections automatiques du port 80 vers le port 443 pour tous les environnements.

Pour les environnements de recette, preprod et prod, il faut créer un certificat Let's Encrypt avec renouvellement automatique. Au moment de la mise en production, nous mettrons en place un certificat authentifié sur la production.

**Supervisor - configuration**

> ; supervisor config file
> [unix_http_server]
> file=/var/run/supervisor.sock   ; (the path to the socket file)
> chmod=0700                       ; sockef file mode (default 0700)
> 
> [supervisord]
> logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
> pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
> childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)
> 
> ; the below section must remain in the config file for RPC
> ; (supervisorctl/web interface) to work, additional interfaces may be
> ; added by defining them in separate rpcinterface: sections
> [rpcinterface:supervisor]
> supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
> 
> [supervisorctl]
> serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket
> 
> ; The [include] section can just contain the "files" setting.  This
> ; setting can list multiple files (separated by whitespace or
> ; newlines).  It can also contain wildcards.  The filenames are
> ; interpreted as relative to this file.  Included files cannot
> ; include files themselves.
> 
> [include]
> ;files = /etc/supervisor/conf.d/*.conf
> files = /var/projects/ayor/*/shared/supervisor-listener.conf

 
**SUDO**

Il faut avoir les droits suffisants pour pouvoir lancer les commandes suivantes :
> sudo /etc/init.d/supervisor stop
> sudo /usr/bin/pkill supervisord
> sudo /etc/init.d/supervisor start

ACL :

Droits 777 par défaut sur :
- /var/projects/ayor/recette/shared/storage/
- /var/projects/ayor/preprod/shared/storage/
- /var/projects/ayor/prod/shared/storage/

Droits 775 par défaut sur :
- /var/projects/ayor/recette/shared/storage/app/
- /var/projects/ayor/preprod/shared/storage/app/
- /var/projects/ayor/prod/shared/storage/app/

Propriétaires/groupes chown -R www-data:dev sur :
- /var/projects/ayor/recette/shared/storage/app/
- /var/projects/ayor/preprod/shared/storage/app/
- /var/projects/ayor/prod/shared/storage/app/

Les utilisateurs www-data, integration_continue et ayor doivent être dans le groupe dev. Ainsi, ils pourront créer/editer des dossiers et fichiers dans le dossier storage/app/

On doit avoir les droits suffisants pour pouvoir utiliser la commande de modification de la CRONTAB de l'utilisateur ayor :
sudo crontab -u ayor -e

Il faut donner la permission aux utilisateurs du groupe dev du serveur de faire des chmod en précisant le chemin absolu vers tous les sous dossiers de /var/projects. Par exemple :
sudo chmod -R 755 /var/projects

**Mailcatcher :**

Création d'un mailcatcher propre à ce projet.

Versions :

	PHP : 8.1.7
	MariaDB : 10.5.15
	Composer : 2.3.7
	npm : 9.6.4
	NodeJS : 18.3.0

Extensions / Packages supplémentaires :

	Supervisor
	PHP SQLite
	Ext Soap
	HTTP2
	libxrender et libxrender-dev (pour wkhtml2pdf)


Variables PHP :

	upload_max_filesize = 50M
	post_max_size = 50M
	max_input_vars = 10000

Variables MySQL :

	max_allowed_packet = 50M`
