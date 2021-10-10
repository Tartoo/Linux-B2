# TP 1 - (re)Familiaration avec un système GNU/Linux MATTHIAS FLAMENT
## 0. Préparation de la machine

### **Setup de deux machines Rocky Linux configurée de façon basique:**
***
## **1. Un accès internet:**
### Une carte réseau dédiée : 
* #### **Commande** : `ip a | ping 8.8.8.8 | ping google.com`
* #### **Résultats** : 
    ![](https://i.imgur.com/DOGvOWR.png)

    ![](https://i.imgur.com/Y310WTI.png)
 
    ![](https://i.imgur.com/ubIJc97.png)

    ![](https://i.imgur.com/olBejyy.png)

La carte Réseau dédiée est la carte enp0s3, la carte enp0s8 étant la carte host-only. De plus, les pings marchent, ce qui prouve un accès a internet.

### Une route par défaut : 
* #### **Commande** : `ip r s`
* #### **Résultats**: 
    ![](https://i.imgur.com/Z0s9foH.png)

    ![](https://i.imgur.com/GvdlUpb.png)

La route par défaut est celle de la carte enp0s3

***

## **2. Un accès à un réseau local (les deux machines peuvent se ping)**
### Carte réseau dédiée (host-only sur VirtualBox):
* #### **Commande** : `ping x.x.x.x`
* #### **Resultat** :
    ![](https://i.imgur.com/L8cgMuR.png)
    
    ![](https://i.imgur.com/0Fy9Ayc.png)
    
***

## **3. Les machines doivent avoir un nom :**
### Changer le nom la machine: 
* #### **Commande** : `sudo nano etc/hostname`
* #### **Resultat** : 
    ![](https://i.imgur.com/M0P3d34.png)

    ![](https://i.imgur.com/fFlM7Oa.png)

***

## **4. Utiliser 1.1.1.1 comme serveur DNS:**
### Configurer le DNS + test:
* #### **Commandes** : `nano /etc/sysconfig/network-scripts/ifcfg-enp0s8` | `dig nom.com`
* #### **Resultat** : 
    ![](https://i.imgur.com/YMfIOjr.png)

    ![](https://i.imgur.com/JZwi4oA.png)

        [tarto@node1 ~]$ dig ynov.com

        ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38217
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 4000
        ;; QUESTION SECTION:
        ;ynov.com.                      IN      A

        ;; ANSWER SECTION:
        🟦    ynov.com.               4448    IN      A       92.243.16.143

        ;; Query time: 2 msec
        🟥    ;; SERVER: 10.33.10.2#53(10.33.10.2)
        ;; WHEN: Wed Sep 22 12:04:14 CEST 2021
        ;; MSG SIZE  rcvd: 53
        ----------------------------------------------------------------------------------
        [tarto@node2 ~]$ dig ynov.com

        ; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2772
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 4000
        ;; QUESTION SECTION:
        ;ynov.com.                      IN      A

        ;; ANSWER SECTION:
        🟦    ynov.com.               4225    IN      A       92.243.16.143

        ;; Query time: 13 msec
        🟥    ;; SERVER: 10.33.10.2#53(10.33.10.2)
        ;; WHEN: Wed Sep 22 12:07:58 CEST 2021
        ;; MSG SIZE  rcvd: 53
        
        🟦:ligne qui contient la réponse : l'IP qui correspond au nom demandé
        🟥:adresse IP du serveur qui m'a répondu
***

## **5. Les machines doivent pouvoir se joindre par leurs noms respectifs:**
### Editer le fichier hosts + tests: 
* #### **Commandes** : `nano /etc/hosts` | `ping "nom de l'autre machine"`
* #### **Resultats** : 
        [tarto@node1 ~]$ cat /etc/hosts
        127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
        ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
        10.101.1.11 node1.tp1.b2 node2.tp1.b2
        ----------------------------------------------------------------------------------
        [tarto@node2 ~]$ cat /etc/hosts
        127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
        ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
        10.101.1.12 node2.tp1.b2 node1.tp1.b2
        ----------------------------------------------------------------------------------
        [tarto@node1 ~]$ ping node2.tp1.b2
        PING node1.tp1.b2 (10.101.1.11) 56(84) bytes of data.
        64 bytes from node1.tp1.b2 (10.101.1.11): icmp_seq=1 ttl=64 time=0.038 ms
        64 bytes from node1.tp1.b2 (10.101.1.11): icmp_seq=2 ttl=64 time=0.110 ms
        64 bytes from node1.tp1.b2 (10.101.1.11): icmp_seq=3 ttl=64 time=0.088 ms
        ^C
        --- node1.tp1.b2 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2044ms
        rtt min/avg/max/mdev = 0.038/0.078/0.110/0.031 ms
        ----------------------------------------------------------------------------------
        [tarto@node2 /]$ ping node1.tp1.b2
        PING node2.tp1.b2 (10.101.1.12) 56(84) bytes of data.
        64 bytes from node2.tp1.b2 (10.101.1.12): icmp_seq=1 ttl=64 time=0.031 ms
        64 bytes from node2.tp1.b2 (10.101.1.12): icmp_seq=2 ttl=64 time=0.032 ms
        64 bytes from node2.tp1.b2 (10.101.1.12): icmp_seq=3 ttl=64 time=0.036 ms
        ^C
        --- node2.tp1.b2 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2085ms
        rtt min/avg/max/mdev = 0.031/0.033/0.036/0.002 ms
        
***

## **6. Le pare-feu est configuré pour bloquer toutes les connexions exceptées celles qui sont nécessaires**
* #### **Commandes** : `sudo sudo firewall-cmd --list-all`
* #### **Resultat** : 
        [tarto@node1 ~]$ sudo firewall-cmd --list-all
        public (active)
          target: default
          icmp-block-inversion: no
          interfaces: enp0s3 enp0s8
          sources:
          services: cockpit dhcpv6-client ssh
          ports:
          protocols:
          masquerade: no
          forward-ports:
          source-ports:
          icmp-blocks:
          rich rules:
        ----------------------------------------------------------------------------------
        [tarto@node2 /]$ sudo firewall-cmd --list-all
        public (active)
          target: default
          icmp-block-inversion: no
          interfaces: enp0s3 enp0s8
          sources:
          services: cockpit dhcpv6-client ssh
          ports:
          protocols:
          masquerade: no
          forward-ports:
          source-ports:
          icmp-blocks:
          rich rules:
          
***

## I. Utilisateurs

## **1. Création et configuration:**
###  Ajouter un utilisateur à la machine, qui sera dédié à son administration. Précisez des options sur la commande d'ajout pour que : 
-    le répertoire home de l'utilisateur soit précisé explicitement, et se trouve dans /home:
        #### Commandes:`sudo useradd Tartofreze --home /home/Tartofreze -s /bin/bash`
        #### Résultats:
            
            [tarto@node1 /]$ sudo useradd Tartofreze --home /home/Tartofreze -s /bin/bash
            Creating mailbox file: File exists
            ---------------------------------------------------------------------------------
            [tarto@node2 /]$ sudo useradd Tartofreze --home /home/Tartofreze -s /bin/bash
            Creating mailbox file: File exists

-    Créer un nouveau groupe admins qui contiendra les utilisateurs de la machine ayant accès aux droits de root via la commande sudo:
        #### Commandes:`sudo groupadd admins | sudo visudo /etc/sudoers |sudo visudo -c`
-    Ajouter votre utilisateur à ce groupe admins:
        #### Commandes:`sudo usermod Tartofreze -a -G admins | groups Tartofreze`
        #### Résultats:
            
            [tarto@node1 /]$ groups Tartofreze
            Tartofreze : Tartofreze admins
            ---------------------------------------------------------------------------------
            [tarto@node2 /]$ groups Tartofreze
            Tartofreze : Tartofreze admins
            
***

## **2. SSH:**
* #### **Commandes** :`ssh-keygen -t rsa -b 4096 | sudo nano /home/tarto/.ssh/authorized_keys | sudo cat /home/tarto/.ssh/authorized_keys`
* #### **Résultats** :
        [tarto@node1 ~]$ sudo cat /home/tarto/.ssh/authorized_key
        [sudo] password for tarto:
        ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDR2txBb2rv7Px12+TuGNIybOlp6Dg0MehF/rfiQMcp1j3KnxAag9Mh4OYaiK3RpKxsCrFdOsRotF6rLSLlMqrDYehfLZC9fIUdJ40yWQGO4Pbk+gJV1gBBdkhONEaYSGCWfZvn+h7ho4V1xTZTI90RTFIEJq0vWtE7Dj1O4n4+gssqGNS7mWl91ZEtgjSORVlIjzQ56K5C5RBdOmmMeitcnX9+Lxfz6th6u8K5uNAFyC9KiBm1LZ9oY6gEUjdRW8rW5Q6dMoXb/w8N1BLRLUXq+CYCQX11PzCIj2BvQevkCA50CpREbEkz6gENQuQbGHJj5CU903D01neYn960NOrSr1NZOraAuZc83wNOn+MwVWWrZWQO9hM0vYepDh9Ug2zJL5pj7zyjSgCQKpPqw+vXV3CVICvX0Xh+5EMh96Vr7DeRpFrdQP+hOQvGY/88xRrXZ1esImARz0TQI8yjyqPvu+ZAT3NzvU51acX+A6MhTZ56guUNaj8JF0xIXFabKiyOnqNC1j57QrDbzTkWwCDK7d73O9p8cQehOHNuqauylFPgbJtYt3qEykbVdWVcc9c1Kq/LrXt7WUThCUHuNYuNlr8zSpuBsZAt3wcznQFNVhzI5/MEOyC09ITr9UDOhsMdCXczPEy9mkpDHCF4gOlF0nkarPhsFUrJsCgzf1k4XQ==mattf@DESKTOP-MN8V18L
        
* #### *Vérification** : 
        PS C:\Users\mattf> ssh tarto@10.101.1.11

        X11 forwarding request failed on channel 0
        Activate the web console with: systemctl enable --now cockpit.socket

        Last login: Sun Sep 26 15:34:13 2021
        [tarto@node1 ~]$

***

## II. Partitionnement

## **1. Préparation de la VM:**
Ajout de disque de 3Go sur node1.

***

## **2. Partitionnement:**
###  Utilisation de LVM : 
*    Agréger les deux disques en un seul volume group:
        #### Commandes:`sudo pvcreate /dev/sdb | sudo pvcreate /dev/sdc | sudo pvs | sudo vgcreate data /dev/sdb | sudo vgextend data /dev/sdc | sudo vgs `
        #### Résultats:
            [tarto@node1 ~]$ sudo pvs
            [sudo] password for tarto:
              PV         VG   Fmt  Attr PSize  PFree
              /dev/sda2  rl   lvm2 a--  <7.00g     0
              /dev/sdb   data lvm2 a--  <3.00g <3.00g
              /dev/sdc   data lvm2 a--  <3.00g <3.00g
         ------------------------------------------------------------------------------------
              [tarto@node1 ~]$ sudo vgs
            [sudo] password for tarto:
              VG   #PV #LV #SN Attr   VSize  VFree
              data   2   0   0 wz--n-  5.99g 5.99g
              rl     1   2   0 wz--n- <7.00g    0
*    Créer 3 logical volumes de 1 Go chacun:
        #### Commandes:`sudo lvcreate -L 1G data -n "nom" | sudo lvs`
        #### Résultats:
            [tarto@node1 ~]$ sudo lvcreate -L 1G data -n first
              Logical volume "first" created.
            [tarto@node1 ~]$ sudo lvcreate -L 1G data -n second
              Logical volume "second" created.
            [tarto@node1 ~]$ sudo lvcreate -L 1G data -n third
              Logical volume "third" created.
         ------------------------------------------------------------------------------------
         [tarto@node1 ~]$ sudo lvs
          LV     VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
          first  data -wi-a-----   1.00g
          second data -wi-a-----   1.00g
          third  data -wi-a-----   1.00g
              root   rl   -wi-ao----  <6.20g
          swap   rl   -wi-ao---- 820.00m
*    Formater ces partitions en ext4:
        #### Commandes:`sudo mkfs -t ext4 /dev/data/"name"`
        #### Résultats:
            [tarto@node1 ~]$ sudo mkfs -t ext4 /dev/data/first
            mke2fs 1.45.6 (20-Mar-2020)
            Creating filesystem with 262144 4k blocks and 65536 inodes
            Filesystem UUID: aab25718-ae36-4c18-bdf5-e3189c1af00e
            Superblock backups stored on blocks:
                    32768, 98304, 163840, 229376

            Allocating group tables: done
            Writing inode tables: done
            Creating journal (8192 blocks): done
            Writing superblocks and filesystem accounting information: done
*    Monter ces partitions pour qu'elles soient accessibles aux points de montage /mnt/part1, /mnt/part2 et /mnt/part3:
        #### Commandes:`sudo mount /dev/data/"name" /mnt/part x | mount | df -h`
        #### Résultats:
            /dev/mapper/data-first on /mnt/part1 type ext4 (rw,relatime,seclabel)
            /dev/mapper/data-second on /mnt/part2 type ext4 (rw,relatime,seclabel)
            /dev/mapper/data-third on /mnt/part3 type ext4 (rw,relatime,seclabel)
            ---------------------------------------------------------------------------------
            [tarto@node1 /]$ df -h
            Filesystem               Size  Used Avail Use% Mounted on
            devtmpfs                 892M     0  892M   0% /dev
            tmpfs                    909M     0  909M   0% /dev/shm
            tmpfs                    909M  8.5M  901M   1% /run
            tmpfs                    909M     0  909M   0% /sys/fs/cgroup
            /dev/mapper/rl-root      6.2G  1.9G  4.4G  30% /
            /dev/sda1               1014M  234M  781M  24% /boot
            tmpfs                    182M     0  182M   0% /run/user/1000
            /dev/mapper/data-first   976M  2.6M  907M   1% /mnt/part1
            /dev/mapper/data-second  976M  2.6M  907M   1% /mnt/part2
            /dev/mapper/data-third   976M  2.6M  907M   1% /mnt/part3
***

###  Faites en sorte que cette partition soit montée automatiquement au démarrage du systèmeUtilisation de LVM : 
* #### **Commande** :`sudo vim /etc/fstab | sudo umount /mnt/partX | sudo mount -av`
* * #### **Resultat** : 
        [tarto@node1 ~]$ sudo umount /mnt/part1
        [tarto@node1 ~]$ sudo umount /mnt/part2
        [tarto@node1 ~]$ sudo umount /mnt/part3
        [tarto@node1 ~]$ sudo mount -av
        /                        : ignored
        /boot                    : already mounted
        none                     : ignored
        mount: /mnt/part1 does not contain SELinux labels.
               You just mounted an file system that supports labels which does not
               contain labels, onto an SELinux box. It is likely that confined
               applications will generate AVC messages and not be allowed access to
               this file system.  For more details see restorecon(8) and mount(8).
        /mnt/part1               : successfully mounted
        mount: /mnt/part2 does not contain SELinux labels.
               You just mounted an file system that supports labels which does not
               contain labels, onto an SELinux box. It is likely that confined
               applications will generate AVC messages and not be allowed access to
               this file system.  For more details see restorecon(8) and mount(8).
        /mnt/part2               : successfully mounted
        mount: /mnt/part3 does not contain SELinux labels.
               You just mounted an file system that supports labels which does not
               contain labels, onto an SELinux box. It is likely that confined
               applications will generate AVC messages and not be allowed access to
               this file system.  For more details see restorecon(8) and mount(8).
        /mnt/part3               : successfully mounted

***

## **III. Gestion de services**

## **1. Interaction avec un service existant:**
* #### **Commande** :`systemctl is-active firewalld ; systemctl is-enabled firewalld`
* #### **Résultat**:
        [tarto@node1 ~]$ systemctl is-active firewalld ; systemctl is-enabled firewalld
        active
        enabled

***
 
## **2. Création de service:**
### **A. Unité simpliste:**
*    Créer un fichier qui définit une unité de service web.service dans le répertoire /etc/systemd/system:
        #### Commandes:`sudo systemctl start web ; sudo systemctl enable web ; sudo systemctl status web`
        #### Résultats:
            [tarto@node1 /]$ sudo systemctl status web
            ● web.service - Very simple web service
               Loaded: loaded (/etc/systemd/system/web.service; enabled; vendor preset: disabled)
               Active: active (running) since Sun 2021-09-26 15:52:39 CEST; 14s ago
             Main PID: 5215 (python3)
            Tasks: 1 (limit: 11407)
               Memory: 10.0M
               CGroup: /system.slice/web.service
                       └─5215 /bin/python3 -m http.server 8888

            Sep 26 15:52:39 node1.tp1.b2 systemd[1]: Started Very simple web service.
         ---------------------------------------------------------------------------------
        ![](https://i.imgur.com/2Dgwj6N.png)

***

### **B. Modification de l'unité:**
* #### **Commande** :`sudo useradd web | curl http://10.101.1.11:8888`
* #### **Résultat** : 
      [tarto@node1 srv]$ curl http://10.101.1.11:8888
        <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"                         "http://www.w3.org/TR/html4/strict.dtd">
        <html>
        <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
        <title>Directory listing for /</title>
        </head>
        <body>
        <h1>Directory listing for /</h1>
        <hr>
        <ul>
        <li><a href="bin/">bin@</a></li>
        <li><a href="boot/">boot/</a></li>
        <li><a href="dev/">dev/</a></li>
        <li><a href="etc/">etc/</a></li>
        <li><a href="home/">home/</a></li>
        <li><a href="lib/">lib@</a></li>
        <li><a href="lib64/">lib64@</a></li>
        <li><a href="media/">media/</a></li>
        <li><a href="mnt/">mnt/</a></li>
        <li><a href="opt/">opt/</a></li>
        <li><a href="proc/">proc/</a></li>
        <li><a href="root/">root/</a></li>
        <li><a href="run/">run/</a></li>
        <li><a href="sbin/">sbin@</a></li>
        <li><a href="srv/">srv/</a></li>
        <li><a href="sys/">sys/</a></li>
        <li><a href="Tartofreze/">Tartofreze/</a></li>
        <li><a href="tmp/">tmp/</a></li>
        <li><a href="usr/">usr/</a></li>
        <li><a href="var/">var/</a></li>
        <li><a href="Web">Web</a></li>
        </ul>
        <hr>
        </body>
        </html>  