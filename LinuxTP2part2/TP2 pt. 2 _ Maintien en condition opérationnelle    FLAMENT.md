# TP2 pt. 2 : Maintien en condition opérationnelle    FLAMENT

# Sommaire

- [TP2 pt. 2 : Maintien en condition opérationnelle](#tp2-pt-2--maintien-en-condition-opérationnelle)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Monitoring](#i-monitoring)
  - [1. Le concept](#1-le-concept)
  - [2. Setup](#2-setup)
- [II. Backup](#ii-backup)
  - [1. Intwo bwo](#1-intwo-bwo)
  - [2. Partage NFS](#2-partage-nfs)
  - [3. Backup de fichiers](#3-backup-de-fichiers)
  - [4. Unité de service](#4-unité-de-service)
    - [A. Unité de service](#a-unité-de-service)
    - [B. Timer](#b-timer)
    - [C. Contexte](#c-contexte)
  - [5. Backup de base de données](#5-backup-de-base-de-données)
  - [6. Petit point sur la backup](#6-petit-point-sur-la-backup)
- [III. Reverse Proxy](#iii-reverse-proxy)
  - [1. Introooooo](#1-introooooo)
  - [2. Setup simple](#2-setup-simple)
  - [3. Bonus HTTPS](#3-bonus-https)
- [IV. Firewalling](#iv-firewalling)
  - [1. Présentation de la syntaxe](#1-présentation-de-la-syntaxe)
  - [2. Mise en place](#2-mise-en-place)
    - [A. Base de données](#a-base-de-données)
    - [B. Serveur Web](#b-serveur-web)
    - [C. Serveur de backup](#c-serveur-de-backup)
    - [D. Reverse Proxy](#d-reverse-proxy)
    - [E. Tableau récap](#e-tableau-récap)

# 0. Prérequis

➜ [TP2 Part.1](../part1/README.md) terminé : on doit avoir un NextCloud et sa base de données dédiée

➜ Machines Rocky Linux

➜ Un unique host-only côté VBox, ça suffira. **L'adresse du réseau host-only sera `10.102.1.0/24`.**

➜ Chaque **création de machines** sera indiqué par **l'emoji 🖥️ suivi du nom de la machine**

➜ Si je veux **un fichier dans le rendu**, il y aura l'**emoji 📁 avec le nom du fichier voulu**. Le fichier devra être livré tel quel dans le dépôt git, ou dans le corps du rendu Markdown si c'est lisible et correctement formaté.

## Checklist

A chaque machine déployée, vous **DEVREZ** vérifier la 📝**checklist**📝 :

- [x] IP locale, statique ou dynamique
- [x] hostname défini
- [x] firewall actif, qui ne laisse passer que le strict nécessaire
- [x] SSH fonctionnel avec un échange de clé
- [x] accès Internet (une route par défaut, une carte NAT c'est très bien)
- [x] résolution de nom
  - résolution de noms publics, en ajoutant un DNS public à la machine
  - résolution des noms du TP, à l'aide du fichier `/etc/hosts`
- [x] monitoring (oui, toutes les machines devront être surveillées)

# I. Monitoring

On bouge pas pour le moment niveau machines :

| Machine         | IP            | Service                 | Port ouvert | IPs autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | 80           | *             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | 3306           | 10.102.1.11             |

## 1. Le concept

[...]

## 2. Setup

De nombreuses solutions de monitoring existent sur le marché. Nous, on va utiliser [Netdata](https://www.netdata.cloud/).  

Maintenant, vous êtes des techs. Alors la page qui vous intéresse encore plus, c'est [le dépôt git de la solution](https://github.com/netdata/netdata).

- le README.md y est souvent très complet
- présente la solution, et les étapes d'install
- fournit les liens vers la doc

🌞 **Setup Netdata**

- y'a plein de méthodes d'install pour Netdata
- on va aller au plus simple, exécutez, sur toutes les machines que vous souhaitez monitorer :

```bash
# Passez en root pour cette opération
$ sudo su -

# Install de Netdata via le script officiel statique
$ bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)

# Quittez la session de root
$ exit
```

🌞 **Manipulation du *service* Netdata**

- un *service* `netdata` a été créé
- déterminer s'il est actif, et s'il est paramétré pour démarrer au boot de la machine
  - si ce n'est pas le cas, faites en sorte qu'il démarre au boot de la machine
  
          #Le service est-il actif:
          [tarto@web ~]$ sudo systemctl status netdata
        ● netdata.service - Real time performance monitoring
           Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; vendor preset: disabled)
           Active: active (running) since Mon 2021-10-11 10:32:41 CEST; 17min ago
           [...]
           ------------------------------------------------------------------------------------------
           [tarto@db ~]$ sudo systemctl status netdata
        ● netdata.service - Real time performance monitoring
           Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; vendor preset: disabled)
           Active: active (running) since Mon 2021-10-11 10:32:41 CEST; 17min ago
           [...]
           ------------------------------------------------------------------------------------------
           #Le service est-il paramétré pour démarrer au boot de la machine:
            [tarto@web ~]$ sudo systemctl is-enabled netdata
            enabled
            [tarto@db ~]$ sudo systemctl is-enabled netdata
            enabled
           
- déterminer à l'aide d'une commande `ss` sur quel port Netdata écoute

        [tarto@web ~]$ ss -ltn | grep 19999
        LISTEN 0      128          0.0.0.0:19999      0.0.0.0:*
        LISTEN 0      128             [::]:19999         [::]:*
        
        [tarto@db ~]$ ss -ltn | grep 19999
        LISTEN 0      128          0.0.0.0:19999      0.0.0.0:*
        LISTEN 0      128             [::]:19999         [::]:*
        
- autoriser ce port dans le firewall

        [tarto@web ~]$ sudo firewall-cmd --add-port=19999/tcp
        Success
        
        [tarto@db ~]$ sudo firewall-cmd --add-port=19999/tcp
        Success

**Eeeeet.... c'est tout !**, rendez-vous sur `http://IP_VM:PORT` pour accéder à l'interface Web de Netdata (depuis un navigateur sur votre PC).  
**C'est sexy na ? Et c'est en temps réel :3**

🌞 **Setup Alerting**

- ajustez la conf de Netdata pour mettre en place des alertes Discord
  - *ui ui c'est bien ça :* vous recevrez un message Discord quand un seul critique est atteint
- [c'est là que ça se passe dans la doc de Netdata](https://learn.netdata.cloud/docs/agent/health/notifications/discord)

        [tarto@web ~]$ sudo cat /etc/netdata/health_alarm_notify.conf
        ###############################################################################
        # sending discord notifications

        # note: multiple recipients can be given like this:
        #                  "CHANNEL1 CHANNEL2 ..."

        # enable/disable sending discord notifications
        SEND_DISCORD="YES"

        # Create a webhook by following the official documentation -
        # https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
        DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/897037085574324244/UBZahljDhBTi-5S7EFyObPEVSWC-ItNmUTg-j4akC0r_0IczhPnBr8JaFbAhGFPpOlCF"

        # if a role's recipients are not configured, a notification will be send to
        # this discord channel (empty = do not send a notification for unconfigured
        # roles):
        DEFAULT_RECIPIENT_DISCORD="alarms"

- vérifiez le bon fonctionnement de l'alerting sur Discord

        sudo bash -x /usr/libexec/netdata/plugins.d/alarm-notify.sh test "sysadmin"

🌞 **Config alerting**

- créez une nouvelle alerte pour recevoir une alerte à 50% de remplissage de la RAM
- testez que votre alerte fonctionne
  - il faudra remplir artificiellement la RAM pour voir si l'alerte remonte correctement
  - sur Linux, on utilise la commande `stress` pour ça

        [tarto@web ~]$ stress --vm 2 --timeout 120#Résultat

    -    Résultat:
![](https://i.imgur.com/A0kOeBc.png)

![[stress test](./pics/stress-test.jpg)](https://gitlab.com/it4lik/b2-linux-2021/-/raw/main/tp/2/part2/pics/stress-test.jpg)

# II. Backup

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |

🖥️ **VM `backup.tp2.linux`**

**Déroulez la [📝**checklist**📝](#checklist) sur cette VM.**

## 1. Intwo bwo

[...]

## 2. Partage NFS

🌞 **Setup environnement**

- créer un dossier `/srv/backup/`
- il contiendra un sous-dossier ppour chaque machine du parc
  - commencez donc par créer le dossier `/srv/backup/web.tp2.linux/`
- il existera un partage NFS pour chaque machine (principe du moindre privilège)

🌞 **Setup partage NFS**

- je crois que vous commencez à connaître la chanson... Google "nfs server rocky linux"
  - [ce lien me semble être particulièrement simple et concis](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=1)

        [tarto@backup ~]]# dnf -y install nfs-utils
        [tarto@backup ~]]# vi /etc/idmapd.conf
        [...]
        Domain = backup.tp2.linux
        [...]
        
        [tarto@backup ~]]# vi /etc/exports
        # create new
        # for example, set [/home/nfsshare] as NFS share
        /svr/backup/web.tp2.linux 10.102.1.0/24(rw,no_root_squash)
        
        [tarto@backup ~]]# systemctl enable --now rpcbind nfs-server
        [tarto@backup ~]]# firewall-cmd --add-service=nfs
        success

🌞 **Setup points de montage sur `web.tp2.linux`**

- [sur le même site, y'a ça](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=2)
- monter le dossier `/srv/backups/web.tp2.linux` du serveur NFS dans le dossier `/srv/backup/` du serveur Web
- vérifier...
  - avec une commande `mount` que la partition est bien montée
  
        sudo mount -t nfs backup.tp2.linux:/srv/backup/web.tp2.linux /srv/backup/
          
  - avec une commande `df -h` qu'il reste de la place

        [tarto@web ~]$ df -h
        Filesystem                                  Size  Used Avail Use% Mounted on
        devtmpfs                                    892M     0  892M   0% /dev
        tmpfs                                       909M  308K  909M   1% /dev/shm
        tmpfs                                       909M  8.5M  901M   1% /run
        tmpfs                                       909M     0  909M   0% /sys/fs/cgroup
        /dev/mapper/rl-root                         6.2G  4.6G  1.7G  74% /
        /dev/sda1                                  1014M  234M  781M  24% /boot
        tmpfs                                       182M     0  182M   0% /run/user/1000
        backup.tp2.linux:/srv/backup/web.tp2.linux  6.2G  1.9G  4.4G  30% /srv/backup
        
  - avec une commande `touch` que vous avez le droit d'écrire dans cette partition
- faites en sorte que cette partition se monte automatiquement grâce au fichier `/etc/fstab`

🌟 **BONUS** : partitionnement avec LVM

- ajoutez un disque à la VM `backup.tp2.linux`
- utilisez LVM pour créer une nouvelle partition (5Go ça ira)
- monter automatiquement cette partition au démarrage du système à l'aide du fichier `/etc/fstab`
- cette nouvelle partition devra être montée sur le dossier `/srv/backup/`

## 3. Backup de fichiers

**Un peu de scripting `bash` !** Le scripting est le meilleur ami de l'admin, vous allez pas y couper hihi.  

La syntaxe de `bash` est TRES particulière, mais ce que je vous demande de réaliser là est un script minimaliste.

Votre script **DEVRA**...

- comporter un shebang
- comporter un commentaire en en-tête qui indique le but du script, en quelques mots
- comporter un commentaire qui indique l'auteur et la date d'écriture du script

Par exemple :

```bash
#!/bin/bash
# Simple backup script
# it4 - 09/10/2021

...
```

🌞 **Rédiger le script de backup `/srv/tp2_backup.sh`**

- le script crée une archive compressée `.tar.gz` du dossier ciblé
  - cela se fait avec la commande `tar`
- l'archive générée doit s'appeler `tp2_backup_YYMMDD_HHMMSS.tar.gz`
  - vous remplacerez évidemment `YY` par l'année (`21`), `MM` par le mois (`10`), etc.
  - ces infos sont déterminées dynamiquement au moment où le script s'exécute à l'aide de la commande `date`
- le script utilise la commande `rsync` afin d'envoyer la sauvegarde dans le dossier de destination
- il **DOIT** pouvoir être appelé de la sorte :

```bash
$ ./tp2_backup.sh <DESTINATION> <DOSSIER_A_BACKUP>
```

📁 **Fichier `/srv/tp2_backup.sh`**

> **Il est strictement hors de question d'utiliser `sudo` dans le contenu d'un script.**  
Il est envisageable, en revanche, que le script doive être lancé avec root ou la commande `sudo` afin d'obtenir des droits élevés pendant son exécution.

🌞 **Tester le bon fonctionnement**

- exécuter le script sur le dossier de votre choix
- prouvez que la backup s'est bien exécutée
- **tester de restaurer les données**
  - récupérer l'archive générée, et vérifier son contenu

🌟 **BONUS**

- faites en sorte que votre script ne conserve que les 5 backups les plus récentes après le `rsync`
- faites en sorte qu'on puisse passer autant de dossier qu'on veut au script : `./tp2_backup.sh <DESTINATION> <DOSSIER1> <DOSSIER2> <DOSSIER3>...` et n'obtenir qu'une seule archive
- utiliser [Borg](https://borgbackup.readthedocs.io/en/stable/) plutôt que `rsync`

## 4. Unité de service

Lancer le script à la main c'est bien. **Le mettre dans une joulie *unité de service* et l'exécuter à intervalles réguliers, de manière automatisée, c'est mieux.**

Le but va être de créer un *service* systemd pour que vous puissiez interagir avec votre script de sauvegarde en faisant :

```bash
$ sudo systemctl start tp2_backup
$ sudo systemctl status tp2_backup
```

Ensuite on créera un *timer systemd* qui permettra de déclencher le lancement de ce *service* à intervalles réguliers.

**La classe nan ?**

![systemd can do that](./pics/suprised-cat.jpg)

---

### A. Unité de service

🌞 **Créer une *unité de service*** pour notre backup

- c'est juste un fichier texte hein
- doit se trouver dans le dossier `/etc/systemd/system/`
- doit s'appeler `tp2_backup.service`
- le contenu :

```bash
[tarto@backup ~]$ cat /etc/systemd/system/tp2_backup.service
[Unit]
Description=Our own lil backup service (TP2)

[Service]
ExecStart=/srv/tp2_backup.sh /srv/backup1  /srv/backup2
Type=oneshot
RemainAfterExit=no

[Install]
WantedBy=multi-user.target
```

> Pour les tests, sauvegardez le dossier de votre choix, peu importe lequel.

🌞 **Tester le bon fonctionnement**

- n'oubliez pas d'exécuter `sudo systemctl daemon-reload` à chaque ajout/modification d'un *service*
- essayez d'effectuer une sauvegarde avec `sudo systemctl start backup`
- prouvez que la backup s'est bien exécutée
  - vérifiez la présence de la nouvelle archive

---

### B. Timer

Un *timer systemd* permet l'exécution d'un *service* à intervalles réguliers.

🌞 **Créer le *timer* associé à notre `tp2_backup.service`**

- toujours juste un fichier texte
- dans le dossier `/etc/systemd/system/` aussi
- fichier `tp2_backup.timer`
- contenu du fichier :

```bash
[Unit]
Description=Periodically run our TP2 backup script
Requires=tp2_backup.service

[Timer]
Unit=tp2_backup.service
OnCalendar=*-*-* *:*:00

[Install]
WantedBy=timers.target
```

> Le nom du *timer* doit être rigoureusement identique à celui du *service*. Seule l'extension change : de `.service` à `.timer`. C'est notamment grâce au nom identique que systemd sait que ce *timer* correspond à un *service* précis.

🌞 **Activez le timer**

- démarrer le *timer* : `sudo systemctl start tp2_backup.timer`
- activer le au démarrage avec une autre commande `systemctl`
- prouver que...
  - le *timer* est actif actuellement
  - qu'il est paramétré pour être actif dès que le système boot

🌞 **Tests !**

- avec la ligne `OnCalendar=*-*-* *:*:00`, le *timer* déclenche l'exécution du *service* toutes les minutes
- vérifiez que la backup s'exécute correctement

---

### C. Contexte

🌞 **Faites en sorte que...**

- votre backup s'exécute sur la machine `web.tp2.linux`
- le dossier sauvegardé est celui qui contient le site NextCloud (quelque part dans `/var/`)
- la destination est le dossier NFS monté depuis le serveur `backup.tp2.linux`
- la sauvegarde s'exécute tous les jours à 03h15 du matin
- prouvez avec la commande `sudo systemctl list-timers` que votre *service* va bien s'exécuter la prochaine fois qu'il sera 03h15

📁 **Fichier `/etc/systemd/system/tp2_backup.timer`**  
📁 **Fichier `/etc/systemd/system/tp2_backup.service`**

## 5. Backup de base de données

Sauvegarder des dossiers c'est bien. Mais sauvegarder aussi les bases de données c'est mieux.

🌞 **Création d'un script `/srv/tp2_backup_db.sh`**

- il utilise la commande `mysqldump` pour récupérer les données de la base de données
- cela génère un fichier `.sql` qui doit ensuite être compressé en `.tar.gz`
- il s'exécute sur la machine `db.tp2.linux`
- il s'utilise de la façon suivante :

```bash
$ ./tp2_backup_db.sh <DESTINATION> <DATABASE>
```

📁 **Fichier `/srv/tp2_backup_db.sh`**  

🌞 **Restauration**

- tester la restauration de données
- c'est à dire, une fois la sauvegarde effectuée, et le `tar.gz` en votre possession, tester que vous êtes capables de restaurer la base dans l'état au moment de la sauvegarde
  - il faut réinjecter le fichier `.sql` dans la base à l'aide d'une commmande `mysql`

🌞 ***Unité de service***

- pareil que pour la sauvegarde des fichiers ! On va faire de ce script une *unité de service*.
- votre script `/srv/tp2_backup_db.sh` doit pouvoir se lancer grâce à un *service* `tp2_backup_db.service`
- le *service* est exécuté tous les jours à 03h30 grâce au *timer* `tp2_backup_db.timer`
- prouvez le bon fonctionnement du *service* ET du *timer*

📁 **Fichier `/etc/systemd/system/tp2_backup_db.timer`**  
📁 **Fichier `/etc/systemd/system/tp2_backup_db.service`**

## 6. Petit point sur la backup

A ce stade vous avez :

- un script qui tourne sur `web.tp2.linux` et qui **sauvegarde les fichiers de NextCloud**
- un script qui tourne sur `db.tp2.linux` et qui **sauvegarde la base de données de NextCloud**
- toutes **les backups sont centralisées** sur `backup.tp2.linux`
- **tout est géré de façon automatisée**
  - les scripts sont packagés dans des *services*
  - les services sont déclenchés par des *timers*
  - tout est paramétré pour s'allumer quand les machines boot (les *timers* comme le serveur NFS)

🔥🔥 **That is clean shit.** 🔥🔥

# III. Reverse Proxy

## 1. Introooooo

Un *reverse proxy* est un outil qui sert d'intermédiaire entre le client et un serveur donné (souvent un serveur Web).

**C'est l'admin qui le met en place, afin de protéger l'accès au serveur Web.**

Une fois en place, le client devra saisir l'IP (ou le nom) du *reverse proxy* pour accéder à l'application Web (ce ne sera plus directement l'IP du serveur Web).

Un *reverse proxy* peut permettre plusieurs choses :

- chiffrement
  - c'est lui qui mettra le HTTPS en place (protocole HTTP + chiffrement avec le protocole TLS)
  - on pourrait le faire directement avec le serveur Web (Apache) dans notre cas
  - pour de meilleures performances, il est préférable de dédier une machine au chiffrement HTTPS, et de laisser au serveur web un unique job : traiter les requêtes HTTP
- répartition de charge
  - plutôt qu'avoir un seul serveur Web, on peut en setup plusieurs
  - ils hébergent tous la même application
  - le *reverse proxy* enverra les clients sur l'un ou l'autre des serveurs Web, afin de répartir la charge à traiter
- d'autres trucs
  - caching de ressources statiques (CSS, JSS, images, etc.)
  - tolérance de pannes
  - ...

---

**Dans ce TP on va setup un reverse proxy NGINX très simpliste.**

![Apache at the back hihi](./pics/nginx-at-the-front-apache-at-the-back.jpg)

## 2. Setup simple

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |
| `front.tp2.linux`  | `10.102.1.14` | Reverse Proxy           | ?           | ?             |

🖥️ **VM `front.tp2.linu`x**

**Déroulez la [📝**checklist**📝](#checklist) sur cette VM.**

🌞 **Installer NGINX**

- vous devrez d'abord installer le paquet `epel-release` avant d'installer `nginx`
  - EPEL c'est des dépôts additionnels pour Rocky
  - NGINX n'est pas présent dans les dépôts par défaut que connaît Rocky
- le fichier de conf principal de NGINX est `/etc/nginx/nginx.conf`

```
[tarto@front ~]$ sudo dnf install epel-release nginx
```

🌞 **Tester !**

- lancer le *service* `nginx`
- le paramétrer pour qu'il démarre seul quand le système boot
- repérer le port qu'utilise NGINX par défaut, pour l'ouvrir dans le firewall
- vérifier que vous pouvez joindre NGINX avec une commande `curl` depuis votre PC

```
[tarto@front ~]$ sudo systemctl start nginx
[tarto@front ~]$ sudo systemctl is-active nginx
active
[tarto@front ~]$ sudo systemctl enable nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
[tarto@front ~]$ sudo systemctl is-enabled nginx
enabled

[tarto@front ~]$ sudo ss -lpnt
State  Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
LISTEN 0       128            0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=3756,fd=8),("nginx",pid=3755,fd=8))
LISTEN 0       128               [::]:80             [::]:*     users:(("nginx",pid=3756,fd=9),("nginx",pid=3755,fd=9))
[tarto@front ~]$ sudo firewall-cmd --add-port 80/tcp
success

C:\Users\mattf>curl 10.102.1.14:80
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
  <head>
    <title>Test Page for the Nginx HTTP Server on Rocky Linux</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <style type="text/css">
    [...]
```

🌞 **Explorer la conf par défaut de NGINX**

- repérez l'utilisateur qu'utilise NGINX par défaut

```
[tarto@front ~]$ sudo cat /etc/nginx/nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
```
- dans la conf NGINX, on utilise le mot-clé `server` pour ajouter un nouveau site
  - repérez le bloc `server {}` dans le fichier de conf principal

```
[tarto@front ~]$ sudo cat /etc/nginx/nginx.conf
 server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
- par défaut, le fichier de conf principal inclut d'autres fichiers de conf
  - mettez en évidence ces lignes d'inclusion dans le fichier de conf principal

```
[tarto@front ~]$ sudo cat /etc/nginx/nginx.conf
# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
```

🌞 **Modifier la conf de NGINX**

- pour que ça fonctionne, le fichier `/etc/hosts` de la machine **DOIT** être rempli correctement, conformément à la **[📝**checklist**📝](#checklist)**

```
[tarto@front ~]$ sudo cat /etc/hosts
10.102.1.11 web.tp2.linux
10.102.1.12 db.tp2.linux
10.102.1.13 backup.tp2.linux
```
- supprimer le bloc `server {}` par défaut, pour ne plus présenter la page d'accueil NGINX

```
[tarto@front ~]$ sudo cat /etc/nginx/nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
```
- créer un fichier `/etc/nginx/conf.d/web.tp2.linux.conf` avec le contenu suivant :
  - j'ai sur-commenté pour vous expliquer les lignes, n'hésitez pas à dégommer mes lignes de commentaires

```bash
[tarto@front ~]$ sudo cat /etc/nginx/conf.d/web.tp2.linux.conf
server {
    listen 80;
    server_name web.tp2.linux;
    location / {

        proxy_pass http://web.tp2.linux;
    }
}
```

# IV. Firewalling

**On va rendre nos firewalls un peu plus agressifs.**

Actuellement je vous ai juste demandé d'autoriser le trafic sur tel ou tel port. C'est bien.

**Maintenant on va restreindre le trafic niveau IP aussi.**

Par exemple : notre base de données `db.tp2.linux` n'est accédée que par le serveur Web `web.tp2.linux`, et par aucune autre machine.  
On va donc configurer le firewall de la base de données pour qu'elle n'accepte QUE le trafic qui vient du serveur Web.

**On va *harden* ("durcir" en français) la configuration de nos firewalls.**

## 1. Présentation de la syntaxe

[...]

## 2. Mise en place

### A. Base de données

🌞 **Restreindre l'accès à la base de données `db.tp2.linux`**

- seul le serveur Web doit pouvoir joindre la base de données sur le port 3306/tcp
- vous devez aussi autoriser votre accès SSH
- n'hésitez pas à multiplier les zones (une zone `ssh` et une zone `db` par exemple)

```
[tarto@db ~]$ sudo firewall-cmd --set-default-zone=drop
success
[tarto@db ~]$ sudo firewall-cmd --new-zone=web --permanent
success
[tarto@db ~]$ sudo firewall-cmd --zone=web --add-source=10.102.1.11/24 --permanent
success
[tarto@db ~]$ sudo firewall-cmd --zone=web --add-port=3306/tcp --permanent
success

[tarto@db ~]$ sudo firewall-cmd --new-zone=ssh --permanent
success
[tarto@db ~]$ sudo firewall-cmd --zone=ssh --add-source=10.102.1.1/24 --permanent
success
[tarto@db ~]$ sudo firewall-cmd --zone=ssh --add-port=22/tcp --permanent
success
```

> Quand vous faites une connexion SSH, vous la faites sur l'interface Host-Only des VMs. Cette interface est branchée à un Switch qui porte le nom du Host-Only. Pour rappel, votre PC a aussi une interface branchée à ce Switch Host-Only.  
C'est depuis cette IP que la VM voit votre connexion. C'est cette IP que vous devez autoriser dans le firewall de votre VM pour SSH.

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

- `sudo firewall-cmd --get-active-zones`
- `sudo firewall-cmd --get-default-zone`
- `sudo firewall-cmd --list-all --zone=?`

```
[tarto@db ~]$ sudo firewall-cmd --get-active-zones
drop
  interfaces: enp0s3 enp0s8
ssh
  sources: 10.102.1.1/24
web
  sources: 10.102.1.11/24
  
[tarto@db ~]$ sudo firewall-cmd --get-default-zone
drop

[tarto@db ~]$ sudo firewall-cmd --list-all --zone=web
web (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.11/24
  services:
  ports: 3306/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
  
[tarto@db ~]$ sudo firewall-cmd --list-all --zone=ssh
ssh (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.1/24
  services:
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
  
[tarto@db ~]$ sudo firewall-cmd --list-all --zone=drop
drop (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

### B. Serveur Web

🌞 **Restreindre l'accès au serveur Web `web.tp2.linux`**

- seul le reverse proxy `front.tp2.linux` doit accéder au serveur web sur le port 80
- n'oubliez pas votre accès SSH

```
[tarto@web ~]$ sudo firewall-cmd --set-default-zone=drop
success
[tarto@web ~]$ sudo firewall-cmd --new-zone=proxy --permanent
success
[tarto@web ~]$ sudo firewall-cmd --zone=proxy --add-source=10.102.1.14/24 --permanent
success
[tarto@web ~]$ sudo firewall-cmd --zone=proxy --add-port=80/tcp --permanent
success

[tarto@web ~]$ sudo firewall-cmd --new-zone=ssh --permanent
success
[tarto@web ~]$ sudo firewall-cmd --zone=ssh --add-source=10.102.1.1/24 --permanent
success
[tarto@web ~]$ sudo firewall-cmd --zone=ssh --add-port=22/tcp --permanent
success
```

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

```
[tarto@web ~]$ sudo firewall-cmd --get-active-zones
drop
  interfaces: enp0s3 enp0s8
proxy
  sources: 10.102.1.14/24
ssh
  sources: 10.102.1.1/24
  
[tarto@web ~]$ sudo firewall-cmd --get-default-zone
drop

[tarto@web ~]$ sudo firewall-cmd --list-all --zone=proxy
proxy (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.14/24
  services:
  ports: 80/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[tarto@web ~]$ sudo firewall-cmd --list-all --zone=ssh
ssh (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.1/24
  services:
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[tarto@web ~]$ sudo firewall-cmd --list-all --zone=drop
drop (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

### C. Serveur de backup

🌞 **Restreindre l'accès au serveur de backup `backup.tp2.linux`**

- seules les machines qui effectuent des backups doivent être autorisées à contacter le serveur de backup *via* NFS
- n'oubliez pas votre accès SSH

```
[tarto@backup ~]$ sudo firewall-cmd --set-default-zone=drop
success
[tarto@backup ~]$ sudo firewall-cmd --new-zone=nfs --permanent
success
[tarto@backup ~]$ sudo firewall-cmd --zone=nfs --add-source=10.102.1.11/24 --permanent
success
[tarto@backup ~]$ sudo firewall-cmd --zone=nfs --add-source=10.102.1.11/22 --permanent
success

[tarto@backup ~]$ sudo firewall-cmd --new-zone=ssh --permanent
success
[tarto@backup ~]$ sudo firewall-cmd --zone=nfs --remove-port=22/tcp --permanent
success
[tarto@backup ~]$ sudo firewall-cmd --zone=ssh --add-port=22/tcp --permanent
success
```

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

```
[tarto@backup ~]$ sudo firewall-cmd --get-active-zones
drop
  interfaces: enp0s3 enp0s8
nfs
  sources: 10.102.1.11/24 10.102.1.12/24
ssh
  sources: 10.102.1.1/24
  
[tarto@backup ~]$ sudo firewall-cmd --get-default-zone
drop

[tarto@backup ~]$ sudo firewall-cmd --list-all --zone=nfs
nfs (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.11/24 10.102.1.12/24
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[tarto@backup ~]$ sudo firewall-cmd --list-all --zone=ssh
ssh (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.1/24
  services:
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[tarto@backup ~]$ sudo firewall-cmd --list-all --zone=drop
drop (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:


```

### D. Reverse Proxy

🌞 **Restreindre l'accès au reverse proxy `front.tp2.linux`**

- seules les machines du réseau `10.102.1.0/24` doivent pouvoir joindre le proxy
- n'oubliez pas votre accès SSH

```
[tarto@front ~]$ sudo firewall-cmd --set-default-zone drop
success
[tarto@front ~]$ sudo firewall-cmd --new-zone=proxy --permanent
success
[tarto@front ~]$ sudo firewall-cmd --zone=proxy --add-source=10.102.1.0/24 --permanent
success
[tarto@front ~]$ sudo firewall-cmd --new-zone=ssh --permanent
success
[tarto@front ~]$ sudo firewall-cmd --zone=ssh --add-source=10.102.1.1/24 --permanent
success
[tarto@front ~]$ sudo firewall-cmd --zone=ssh --add-port=22/tcp --permanent
success
```

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

```
[tarto@front ~]$ sudo firewall-cmd --get-active-zones
drop
  interfaces: enp0s3 enp0s8
proxy
  sources: 10.102.1.0/24
ssh
  sources: 10.102.1.1/24
  
[tarto@front ~]$ sudo firewall-cmd --get-default-zone
drop

[tarto@front ~]$ sudo firewall-cmd --list-all --zone=proxy
proxy (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.0/24
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[tarto@front ~]$ sudo firewall-cmd --list-all --zone=ssh
ssh (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 10.102.1.1/24
  services:
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[tarto@front ~]$ sudo firewall-cmd --list-all --zone=drop
drop (active)
  target: DROP
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

### E. Tableau récap

🌞 **Rendez-moi le tableau suivant, correctement rempli :**

| Machine            | IP            | Service                 | Port ouvert         | IPs autorisées                                    |
|--------------------|---------------|-------------------------|---------------------|---------------------------------------------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | `80/tcp` `22/tcp`   | `10.102.1.14/24` `10.102.1.1/24`                  |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | `3306/tcp` `22/tcp` | `10.102.1.11/24` `10.102.1.1/24`                  |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | `22/tcp`            | `10.102.1.11/24` `10.102.1.12/24` `10.102.1.1/32` |
| `front.tp2.linux`  | `10.102.1.14` | Reverse Proxy           | `80/tcp` `22/tcp`   | `10.102.1.0/24` `10.102.1.1/24`                   |