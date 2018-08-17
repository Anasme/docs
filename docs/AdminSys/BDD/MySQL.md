# MYSQL

## Le fichier de configuration

> le fichier de configuration est le fichier **my.cnf** qui se trouve habituellement dans **/etc/mysql/** sinon dans **/etc/mysql/mariadb.conf.d** pour mariaDB

### Grandes lignes my.cnf

- le port d'écoute de **MySQLd** : **3306**
- différents dossiers utilisés par **MySQLd**
- le nom d'utilisateur des processus de **MySQLd** (qui est **mysql**)
- des paramètres liés à la performance (taille mémoire allouée, taille du cache ...)
- les fichiers de log 

### Fichier de log

Dans my.cnf décommenter :

- `general_log_file = /var/log/mysql/mysql.log`
- `general_log = 1`

!!! warning
    logs très utiles mais consommateurs de ressources :arrow_right: tueurs de perfs.
    Ne pas hesiter à les activer pour du test mais désactiver en production !



### Changer le répertoire de logs MyQSL

> Nécessaire si trop d'IO sur le disque (log + datas)

my.cnf : 

- `log-bin=/'chemindunouveaurépertoire'/mysql-bin` 

!!! info
    Purger un maximum de logs (PURGE BINLOG...) pour ne pas avoir trop de fichiers à transférer.

***Arrêter le service*** 

```bash
service mysql stop
```

***Déplacer les fichiers vers le nouveau répertoire*** 

```BASH
mv /var/lib/mysql /mysql-bin.* /chemindunouveaurépertoire/mysql-bin
```

***Redémarrer le service***

```bash
 service mysql start
```

### La directive bind-address

> Cette directive sert à restreindre l'accès au service MySQLd. Seule la machine (et donc les logiciels outils installés dessus) indiquée dans cette directive peut accéder au service.

***Si on veut accéder que par la machine local (localhost) :***

```bash
bind-address = 127.0.0.1
```

***Si on veut uniquement un serveur distant :*** 

```bash
bind-address = "ip du serveur"
```

***Si on veut ouvrir à tous les serveurs :*** 

```bash
bind-address = 0.0.0.0
```

!!! info
    Il n'y a pas de demie mesure, la sécurisation doit se faire en amont pour les serveur de données.

## Sécurisation du service MySQL

```bash
mysql_secure_installation
```

**Cela permet de :** 

- s'assurer de bien avoir un mot de passe root MySQL 
- empêcher d'accéder en root MySQL depuis le réseau (par le socket MySQL ; par contre on accède à distance par SSH et ensuite on se connecte à SQL une fois rentré dans la machine donc en « local ».
- empêcher les connexions anonymes
- supprimer la base de test (accessible par défaut par tous, même anonymes) et les privilèges qui autorisent tout le monde à accéder à toutes les bases commençant par test_

## Commandes MySQL (base)

***Se connecter***

```bash
mysql -h 'host '-u 'user' -p 
```

- `-h` premet de préciser l'hôte (host)
- `-u` permet de préciser l'utilisateur (user)
- `-p` indique que nous allons ensuite taper le mot de passe (password)

***Lister les bases***

```mysql
SHOW DATABASES;
```

***Selectionner une base***

```mysql
USE "nom de la bdd";
```

***Lister les tables d'une base***

- si base selectionné

```mysql 
SHOW TABLES;
```

- sinon

```mysql 
SHOW TABLES FROM "nom de la bdd";
```
***Créer une base*** 

```mysql
CREATE DATABASE "nom bdd";
```

***Créer un utilisateur*** 

```mysql
CREATE USER 'user'@'localhost';
```

***Affecter une mot de passe à un utilisateur***

```mysql
SET PASSWORD FOR 'user' = PASSWORD('mot de passe');
```

***Donner tout les droits à un utilisateur sur un BD (voir "Les privilèges" pour les types de droits)***

```mysql
GRANT ALL PRIVILEGES ON 'BASE.*' TO user@localhost IDENTIFIED by 'mot de passe';
```

***Revoquer les droits d'un utilisateur***

```mysql
REVOKE ALL PRIVILEGES ON 'BASE.*' FROM user;
```

***Supprimer un compte utilisateur***

```mysql
DROP user 'test'@'localhost';
```

## Les privilèges (droits)

!!!info
    En matière de bases de données, on parle de ***privilèges*** pour parler des *droits*. Ces privilèges s'appliquent à des comptes. Tel compte à le droit de faire telle action.

**Les information des utilisateurs et des privilèges sont stockées dans la base de données mysql :**

- table user ➡️ utilisateurs (privilèges globaux) 

### Quatre tables pour stocker les privilèges users  

- `db` : privilèges au niveau des bases de données. 
- `tables_priv` : privilèges au niveau des tables. 
- `columns_priv` : privilèges au niveau des colonnes. 
- `procs_priv` : privilèges au niveau des routines (procédures et fonctions stockées).

### Droits principaux

- `GRANT` : le droit de donner des droits
- `CREATE` : Créer des utilisateurs, des bases, des tables ou des index
- `DROP` : Supprimer des utilisateurs, des bases, des tables ou des index
- `ALTER` : modifier la structure de tables
- `DELETE` : Supprimer des données
- `INSERT` : Ajouter des données
- `UPDATE` : Modifier des données

!!!example 

    - Ajouter le droit de faire un INSERT sur la table « table1 » pour l’utilisateur “invite” : 
    
    ```mysql
    GRANT INSERT ON table1 TO invite;
    ```
    
    - Attribuer tous les droits à l’utilisateur “invite” sur la base de données utilisée : 
    
    ```mysql
    GRANT ALL ON * TO invite;
    ```
    
    - Retirer le droit de faire un INSERT sur toutes les tables de la BDD utilisée pour l’utilisateur « invite » :


    ```mysql
    REVOKE INSERT ON * FROM invite;
    ```
    
    -  Retirer le droit de faire un INSERT sur la table « table1 » pour l’utilisateur “invite” : 


    ```mysql
    REVOKE INSERT ON table1 FROM invite;
    ```
    
    - Retirer tous les droits à l’utilisateur “invite” sur toutes les tables de la BDD utilisée : 


    ```mysql
    REVOKE ALL PRIVILEGES ON * FROM invite
    ```

!!! danger
    ne jamais donner **WITH GRANT OPTION** à un utilisateur non         administrateur système 

## Supervision

### MySQLTuner

> Script **PERL** qui permet d'optimiser les performances d'un serveur de bases de données MySQL en faisant un diagnostic, voir: https://github.com/major/MySQLTuner-perl

### Mytop 

> Outil "top like" écrit en PERL mais pour les base de données. (intégré à deb9)

***utilisation*** 

```bash
mytop --prompt -d "base de donnée"
```

***Si cela ne fonctionne pas , installer les dépendances PERL suivante :***

```bash
 apt-get install libconfig-inifiles-perl
```

**Mysql only :**

fichier de confg d'exemple dans /usr/share /doc/mysql-server-5.X.XX

## Tuning

***key_buffer*** 

```mysql
SHOW GLOBAL STATUS
```

ligne `key_reads/key_read_requests` radio < 0.01 pour éviter accès disque 

sinon augmenter le `key_buffer`

***thread_concurrency (machine multicoeur)***

Valeur à x2 nombre de coeurs

***Activer le log des requêtes lentes*** 

`slow_query_log=1` 

`slow_query_log_file=/var/log/mysql/mysql-slow.log` 

***Et demander d'indiquer le temps le plus lent***

`long_query_time = 2` 

***Analyser les résultats***

```bash
tail -n 1000 /var/log/mysql/mysql-slow.log 
```
***check "connection leakage"***

```mysql
SHOW PROCESSLIST;
```
***Analyser les verrous***

```mysql
SHOW STATUS LIKE 'Table%';
SHOW ENGINE INNODB STATUS (pour innodb)
```
## Sauvegarde

### mysqldump (à chaud)

***bases précises***

```bash
 mysqldump --user=root -p --databases mysql wiki > wiki_bd_backup.sql
```
***all base***

```bash
mysqldump --user=root -p --all-databases | gzip > save.mysql.sql.gz
```
### backup à froid : 

>  Elle consiste à sauvegarder les fichiers “physiques” du serveur de données. Elle nécessite, par définition l’arrêt du serveur. 

Aucune activité n’est donc possible. C’est un avantage pour le DBA mais il est parfois difficile voire impossible de stopper l’activité en cours. En revanche si l’on dispose d’un réplica on peut tout à fait l’envisager. Celui ci se “re synchronisera” à son redémarrage. La sauvegarde à froid consiste à archiver certains fichiers du serveur`**/var/lib/mysql/**` et `**/var/log/mysql.log**` 

Pour les tables MyISAM il s’agit plus précisement des fichiers `.FRM`, `.MYI` et `.MYD`. Pour les tables InnoDB : `.FRM`, les fichiers de données (ibdata), les fichiers des tables .ibd (si vous êtes en innodb_file_per_table) et les fichiers journaux (iblogfile) intermédiaires.

## Restauration

***dump en selectionnant la base***

```mysql
use database;
source /chemin/du/fichier/backup.sql
```

***ou pour le scripting*** 

```bash
mysql -u myuser -p < /chemin/du/fichier/backup.sql ( automatisation totale avec --password = < mot de passe >
```

!!!info
    pour restaurer à froid, il suffit de couper le service et de déposer les fichiers aux bons emplacements

## Automatisation des sauvegardes

***automysqlbackup, Script "tout en un"***

- Notification par mail
- Compression et chiffrement des sauvegardes
- Rotation des sauvegardes configurable 
- Sauvegardes incrémentales

```bash
apt-get install automysqlbackup
```

à configure dans  `/etc/default/automysqlbackup`

***rsnapshot***

```bash
apt-get install rsnapshot
```

voir https://wiki.debian-fr.xyz/Rsnapshot

***outil de backup à chaud***

- https://www.percona.com/doc/percona-xtrabackup/LATEST/index.html

## Migration Mysql vers MariaDB (deb<9)

***Maj du serveur Debian***

```bash
apt-get update && apt-get upgrade
apt-get install python-software-properties software-properties-common
```
***Ajout des dépot MariaDB voir*** 

- https://downloads.mariadb.org/mariadb/repositories/

***update des paquets et installe***

```bash
apt-get update && apt-get install mariadb-server
```

!!!info 
    Si Mysql > 5.5 migration directement sur 10.1 sinon 5.5

## Cluser MariaDB/Galera

> Cluster synchro en maître-maître

!!!info
    En production favoriser 3 serveurs minimum

Création du cluster

```bash
service mysql stop
mysqld --wsrep-new-cluster
#or
galera_new_cluster
```
sur les autres noeuds

```bash
service mysql stop
mysqld --wsrep_cluster_address=gcomm://ip(ou nom DNS du serveur qui a créé le cluster)
```
***voir le statut de la syncrho***

```mysql
SHOW STATUS LIKE 'wsrep_%'
```

!!!tip
    configuration affinée voir http://www.severalnines.com/New-Galera-Configurator/index.html

## Troubleshoot

### Table corrompue

***Message d'erreur*** 

> “Table is marked as crashed and should be repaired” 

***Solution***

```mysql
repair table bd.tablename;
```

### Récupération de mot de passe root MySQL

***arrêt du service et redémarrage en mode rescue***

```bash 
systemctl stop mysql && mysqld_safe --skip-grant-tables & 
```

***Connexion en root*** 

```bash
mysql -u root
```
***dans la console mysql***
```mysql
#select base
mysql> use mysql; 
#change password
mysql> update user set password=PASSWORD("nouveaumotdepasse") where user='root'; 
#save
mysql> flush privileges; 
#leave
mysql> quit 
```

***arrêt du mode rescue***

```bash
systemctl stop mysql
```

***test de connexion***

```bash
mysql -u root -p
```
