# TP2 pt. 1 : Gestion de service        FLAMENT

# I. Un premier serveur web

## 1. Installation

**Installer le serveur Apache**

- paquet `httpd`

        sudo dnf install -y httpd
        
- la conf se trouve dans `/etc/httpd/`
  - le fichier de conf principal est `/etc/httpd/conf/httpd.conf`
  - je vous conseille **vivement** de virer tous les commentaire du fichier, à défaut de les lire, vous y verrez plus clair
    - avec `vim` vous pouvez tout virer avec `:g/^ *#.*/d`

> Ce que j'entends au-dessus par "fichier de conf principal" c'est que c'est **LE SEUL** fichier de conf lu par Apache quand il démarre. C'est souvent comme ça : un service ne lit qu'un unique fichier de conf pour démarrer. Cherchez pas, on va toujours au plus simple. Un seul fichier, c'est simple.  
**En revanche** ce serait le bordel si on mettait toute la conf dans un seul fichier pour pas mal de services.  
Donc, le principe, c'est que ce "fichier de conf principal" définit généralement deux choses. D'une part la conf globale. D'autre part, il inclut d'autres fichiers de confs plus spécifiques.  
On a le meilleur des deux mondes : simplicité (un seul fichier lu au démarrage) et la propreté (éclater la conf dans plusieurs fichiers).

🌞 **Démarrer le service Apache**

- le service s'appelle `httpd` (raccourci pour `httpd.service` en réalité)
  - démarrez le
  - faites en sorte qu'Apache démarre automatique au démarrage de la machine
  - ouvrez le port firewall nécessaire
    - utiliser une commande `ss` pour savoir sur quel port tourne actuellement Apache
    - [une petite portion du mémo est consacrée à `ss`](https://gitlab.com/it4lik/b2-linux-2021/-/blob/main/cours/memo/commandes.md#r%C3%A9seau)
    
            State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port         Process
            LISTEN          0               128                            0.0.0.0:22                          0.0.0.0:*             users:(("sshd",pid=819,fd=5))
            LISTEN          0               128                                  *:80                              *:*             users:(("httpd",pid=2557,fd=4),("httpd",pid=2556,fd=4),("httpd",pid=2555,fd=4),("httpd",pid=2553,fd=4))
            LISTEN          0               128                               [::]:22                             [::]:*             users:(("sshd",pid=819,fd=7))

**En cas de problème** (IN CASE OF FIIIIRE) vous pouvez check les logs d'Apache :

```bash
# Demander à systemd les logs relatifs au service httpd
$ sudo journalctl -xe -u httpd

# Consulter le fichier de logs d'erreur d'Apache
$ sudo cat /var/log/httpd/error_log

# Il existe aussi un fichier de log qui enregistre toutes les requêtes effectuées sur votre serveur
$ sudo cat /var/log/httpd/access_log
```

🌞 **TEST**

- vérifier que le service est démarré

        [tarto@web ~]$ sudo systemctl status httpd
        ● httpd.service - The Apache HTTP Server
           [...]
           Active: active (running) since Sat 2021-10-05 15:38:47 CEST; 46min 12s ago
           [...]
        
- vérifier qu'il est configuré pour démarrer automatiquement

        [tarto@web ~]$ sudo systemctl is-active httpd ; sudo systemctl is-enabled httpd
        active
        enabled
        
- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement

        [tarto@web ~]$ curl localhost
        <!doctype html>
            <html>
              <head>
              [...]
              [...]
              </footer>

            </body>
        </html>
        
- vérifier avec votre navigateur (sur votre PC) que vous accéder à votre serveur web
    ![](https://i.imgur.com/ucdiqOm.png)


## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

- donnez la commande qui permet d'activer le démarrage automatique d'Apache quand la machine s'allume

        sudo systemctl enable httpd
        
- prouvez avec une commande qu'actuellement, le service est paramétré pour démarré quand la machine s'allume

        sudo systemctl is-enabled httpd
        
- affichez le contenu du fichier `httpd.service` qui contient la définition du service Apache

        [tarto@web ~]$ cat /usr/lib/systemd/system/httpd.service
        # See httpd.service(8) for more information on using the httpd service.

        # Modifying this file in-place is not recommended, because changes
        # will be overwritten during package upgrades.  To customize the
        # behaviour, run "systemctl edit httpd" to create an override unit.

        # For example, to pass additional options (such as -D definitions) to
        # the httpd binary at startup, create an override unit (as is done by
        # systemctl edit) and enter the following:

        #       [Service]
        #       Environment=OPTIONS=-DMY_DEFINE

        [Unit]
        Description=The Apache HTTP Server
        Wants=httpd-init.service
        After=network.target remote-fs.target nss-lookup.target httpd-        init.service
        Documentation=man:httpd.service(8)
        
        [Service]
        Type=notify
        Environment=LANG=C

        ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
        ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
        # Send SIGWINCH for graceful stop
        KillSignal=SIGWINCH
        KillMode=mixed
        PrivateTmp=true

        [Install]
        WantedBy=multi-user.target
🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

- mettez en évidence la ligne dans le fichier de conf qui définit quel user est utilisé

        sudo cat /etc/httpd/conf/httpd.conf
        ServerRoot "/etc/httpd"

        Listen 8080

        Include conf.modules.d/*.conf

        User apache
        Group apache
        
        [...]
- utilisez la commande `ps -ef` pour visualiser les processus en cours d'exécution et confirmer que apache tourne bien sous l'utilisateur mentionné dans le fichier de conf

        [tarto@web ~]$ ps -ef
        apache    2554    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        apache    2555    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        apache    2556    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        apache    2557    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        
- vérifiez avec un `ls -al` le dossier du site (dans `/var/www/...`) et tout son contenu appartient au même utilisateur

        [tarto@web /]$ ls -al /var/www/
        total 4
        drwxr-xr-x.  4 root root   33 Sep 29 12:26 .
        drwxr-xr-x. 21 root root 4096 Sep 29 12:26 ..
        drwxr-xr-x.  2 root root    6 Jun 11 17:35 cgi-bin
        drwxr-xr-x.  2 root root    6 Jun 11 17:35 html
        
🌞 **Changer l'utilisateur utilisé par Apache**

- créez le nouvel utilisateur
  - pour les options de création, inspirez-vous de l'utilisateur Apache existant
    - le fichier `/etc/passwd` contient les informations relatives aux utilisateurs existants sur la machine
    - servez-vous en pour voir la config actuelle de l'utilisateur Apache par défaut
- modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur
- redémarrez Apache
- utilisez une commande `ps` pour vérifier que le changement a pris effet
        
        [tarto@web ~]$ ps -ef
        OvSacre+    2554    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        OvSacre+    2555    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        OvSacre+    2556    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        OvSacre+    2557    2553  0 15:24 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
        
🌞 **Faites en sorte que Apache tourne sur un autre port**

- modifiez la configuration d'Apache pour lui demande d'écouter sur un autre port
- ouvrez un nouveau port firewall, et fermez l'ancien
- redémarrez Apache
- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi

        [tarto@web ~]$ ss -alnpt
        State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       Process
        LISTEN        0             128                        0.0.0.0:22                      0.0.0.0:*
        ✔️LISTEN        0             128                              *:8080                          *:*
        LISTEN        0             128                           [::]:22                         [::]:*
        
- vérifiez avec `curl` en local que vous pouvez joindre Apache sur le nouveau port

        [tarto@web ~]$ curl localhost:8080
        <!doctype html>
            <html>
              <head>
              [...]
              [...]
              </footer>

            </body>
        </html>
        
- vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port

    ![](https://i.imgur.com/VFpUw4F.png)


📁 **Fichier `/etc/httpd/conf/httpd.conf`** (que vous pouvez renommer si besoin, vu que c'est le même nom que le dernier fichier demandé)

# II. Une stack web plus avancée

## 1. Intro

[...]

## 2. Setup

### A. Serveur Web et NextCloud

**Créez les 2 machines et déroulez la [📝**checklist**📝](#checklist).**

🌞 Install du serveur Web et de NextCloud sur `web.tp2.linux`

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/cms/cloud_server_using_nextcloud/#next-steps)
  - **uniquement pour le serveur Web + NextCloud**, vous ferez la base de données MariaDB après
  - quand ils parlent de la base de données, juste vous sautez l'étape, on le fait après :)
- je veux dans le rendu **toutes** les commandes réalisées
  - n'oubliez pas la commande `history` qui permet de voir toutes les commandes tapées précédemment

         #Installation et configurations des repos             
         sudo dnf install epel-release
         dnf update
         sudo dnf update
         sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
         sudo dnf module list php
         sudo dnf module enable php:remi-7.4
         sudo dnf module list php
         php remi-7.4 [e] common [d], devel, minimal PHP scripting language
         sudo dnf module list php
         
         #Installation des packages         
         sudo dnf install -y httpd vim wget zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp
         
         #Configurations des packages et fichiers  
         sudo systemctl enable httpd
         sudo mkdir /etc/httpd/sites-available
         sudo vim /etc/httpd/sites-available/com.web.nextcloud
         sudo mkdir /etc/httpd/sites-enabled
         sudo ln -s /etc/httpd/sites-available/com.web.nextcloud /etc/httpd/sites-enabled/
         sudo mkdir -p /var/www/sub-domains/com.web.com/html
         sudo vim /etc/httpd/conf/httpd.conf
         timedatectl
         sudo vim /etc/opt/remi/php74/php.ini
         ls -al /etc/localtime
         
         #Installation de NextCloud
         sudo wget https://download.nextcloud.com/server/releases/nextcloud-22.2.0.zip
         sudo unzip nextcloud-22.2.0.zip
         sudo cp -Rf * /var/www/sub-domains/com.web.nextcloud/html/
         sudo chown -Rf apache.apache /var/www/sub-domains/com.web.nextcloud/html
         sudo systemctl restart httpd

Une fois que vous avez la page d'accueil de NextCloud sous les yeux avec votre navigateur Web, **NE VOUS CONNECTEZ PAS** et continuez le TP

📁 **Fichier `/etc/httpd/conf/httpd.conf`**  
📁 **Fichier `/etc/httpd/conf/sites-available/web.tp2.linux`**

### B. Base de données

🌞 **Install de MariaDB sur `db.tp2.linux`**

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/database/database_mariadb-server/)
- manipulation 
- je veux dans le rendu **toutes** les commandes réalisées
    
        #installtion de mariaDB
        
        sudo dnf install -y mariadb-server
        sudo systemctl enable mariadb
        sudo systemctl restart mariadb
        mysql_secure_installation
        
- vous repérerez le port utilisé par MariaDB avec une commande `ss` exécutée sur `db.tp2.linux`

        [tarto@db ~]$ sudo ss -lpnt
        State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
        LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=825,fd=5))
        LISTEN     0          128                     [::]:22                   [::]:*        users:(("sshd",pid=825,fd=7))
        ✔️LISTEN     0          80                         *:3306                    *:*        users:(("mysqld",pid=911,fd=21))

🌞 **Préparation de la base pour NextCloud**

- une fois en place, il va falloir préparer une base de données pour NextCloud :
  - connectez-vous à la base de données à l'aide de la commande `sudo mysql -u root`
  - exécutez les commandes SQL suivantes :

```sql
[tarto@db ~]$ mysql -u root -p

MariaDB [(none)]> CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'meow';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)
```

> Par défaut, vous avez le droit de vous connectez localement à la base si vous êtes `root`. C'est pour ça que `sudo mysql -u root` fonctionne, sans nous demander de mot de passe. Evidemment, n'importe quelles autres conditions ne permettent pas une connexion aussi facile à la base.

🌞 **Exploration de la base de données**

- afin de tester le bon fonctionnement de la base de données, vous allez essayer de vous connecter, comme NextCloud le fera :
  
        #Ouverture du port
        [tarto@db ~]$ sudo firewall-cmd --add-port=3306/tcp
        success
        [tarto@db ~]$
      
        #Connection depuis web.tp2
        [tarto@web ~]$ mysql -u nextcloud -h 10.102.1.12 -p
        Enter password:
        Welcome to the MariaDB monitor.  Commands end with ; or \g.
        Your MariaDB connection id is 11
        Server version: 10.3.28-MariaDB MariaDB Server

        Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

        Type 'help;' or '\h' for help. Type '\c' to clear the current input         statement.

        #Exploration de la Base de données
        MariaDB [(none)]> SHOW DATABASES;
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | nextcloud          |
        +--------------------+
        2 rows in set (0.002 sec)

        MariaDB [(none)]> USE nextcloud;
        Database changed
        MariaDB [none]> SHOW TABLES;
        Empty set (0.000 sec)

        #Liste de tous les utilisateurs de la db
        MariaDB [none]> SELECT user FROM mysql.user;
        +-----------------------+-------------+
        | user                  | host        |
        +-----------------------+-------------+
        | nextcloud@10.102.1.11 | %           |
        | nextcloud             | 10.102.1.11 |
        | root                  | 127.0.0.1   |
        | root                  | ::1         |
        | root                  | localhost   |
        +-----------------------+-------------+
        5 rows in set (0.000 sec)

### C. Finaliser l'installation de NextCloud

🌞 sur votre PC

- modifiez votre fichier `hosts` (oui, celui de votre PC, de votre hôte)
  - pour pouvoir joindre l'IP de la VM en utilisant le nom `web.tp2.linux`
  
          PS C:\Windows\System32\drivers\etc> cat .\hosts
          [...]
          10.102.1.11     web.tp2.linux

- avec un navigateur, visitez NextCloud à l'URL `http://web.tp2.linux`
  - c'est possible grâce à la modification de votre fichier `hosts`
  - ![](https://i.imgur.com/xXZTo34.jpg)

- on va vous demander un utilisateur et un mot de passe pour créer un compte admin
  - ne saisissez rien pour le moment
- cliquez sur "Storage & Database" juste en dessous
  - choisissez "MySQL/MariaDB"
  - saisissez les informations pour que NextCloud puisse se connecter avec votre base
- saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation
- ![](https://i.imgur.com/ss8yonu.png)


🌞 **Exploration de la base de données**

- connectez vous en ligne de commande à la base de données après l'installation terminée
- déterminer combien de tables ont été crées par NextCloud lors de la finalisation de l'installation
  - ***bonus points*** si la réponse à cette question est automatiquement donnée par une requête 
  
        MariaDB [(none)]> USE nextcloud;
        Reading table information for completion of table and column names
        You can turn off this feature to get a quicker startup with -A

        Database changed
        MariaDB [nextcloud]> SELECT COUNT(*) from INFORMATION_SCHEMA.TABLES
            -> WHERE TABLE_TYPE = 'BASE TABLE' ;
        +----------+
        | COUNT(*) |
        +----------+
        |      191 |
        +----------+
        1 row in set (0.009 sec)
        
        191 tables ont donc été crées.
  
  
  
  
  
  
  
  

### Tableau:

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | 80/tcp          | *             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | 3306/tcp          | 10.102.1.11            |