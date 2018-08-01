

# Serveur de données 

[TOC]

## Environnement réseau

La base de donné est un élement critique et sensible d'un architecture pour ces 2 raisons:

> - Il n'est pas conçu pour se protéger efficacement des attaques
> - il contient des données, c'est à dire la seule information pérenne dans notre architecture

C'est pou cela qu'il faut l'isoler au maximum : 

### *à ne pas faire** :

![https://anasme.github.io/images/DMZ_nottodo.png](https://anasme.github.io/images/DMZ_nottodo.png)

- ### **à ne pas faire non plus** :

![https://anasme.github.io/images/DMZ_nottodo.png](https://anasme.github.io/images/DMZ_nottodotoo.png)

- ### **bien de faire** :

![img](https://anasme.github.io/images/Untitled%20Diagram.png)

- ### **meilleur chose à faire** :

![img](https://anasme.github.io/images/besttopdo.png)





## Configuration MySQL

### Le fichier de configuration

> le fichier de configuration est le fichier **my.cnf** qui se trouve habituellement dans **/etc/mysql/** sinon dans **/etc/mysql/mariadb.conf.d** pour mariaDB

##### grandes lignes my.cnf

- le port d'écoute de **MySQLd** : qui est le port **3306**
- différents dossiers utilisés par **MySQLd**
- le nom d'utilisateur des processus de **MySQLd** (qui est **mysql**)
- des paramètres liés à la performance (taille mémoire allouée, taille du cache ...) ou à [la configuration](https://elearning.u-pem.fr/mod/book/view.php?id=37531) fine de moteurs isam, innodb
- les fichiers de log 

### Fichier de log

dans my.cnf décommenter :

- general_log_file        = /var/log/mysql/mysql.log
- general_log             = 1

:warning: logs très utiles mais consommateurs de ressources :arrow_right: tueurs de perfs.

:arrow_right_hook: ne pas hesiter à les activer pour du test mais désactiver en production !

##### Changer le répertoire de logs MyQSL

> Nécessaire si trop d'IO sur le disque (log + datas)

my.cnf : 

- log-bin=/'chemindunouveaurépertoire'/mysql-bin 

:information_source: Purger un maximum de logs (PURGE BINLOG...) pour ne pas avoir trop de fichiers à transférer.

Arrêter le service :

```bash
service mysql stop
```

- Déplacer les fichiers vers le nouveau répertoire :

```BASH
mv /var/lib/mysql /mysql-bin.* /chemindunouveaurépertoire/mysql-bin
```

- Redémarrer le service

```bash
 service mysql start
```

#### La directive bind-address

> Cette directive sert à restreindre l'accès au service MySQLd. Seule la machine (et donc les logiciels outils installés dessus) indiquée dans cette directive peut accéder au service.

Si on veut accéder que par la machine local (localhost) :

```bash
bind-address = 127.0.0.1
```

Si on veut uniquement un serveur distant : 

```bash
bind-address = "ip du serveur"
```

Si on veut ouvrir à tous les serveurs : 

```bash
bind-address = 0.0.0.0
```

ℹ️ Il n'y a pas de demie mesure, la sécurisation doit se faire en amont pour les serveur de données.

#### Sécurisation du service MySQL

```bash
#mysql_secure_installation
```

Cela permet de : • s'assurer de bien avoir un mot de passe root MySQL • empêcher d'accéder en root MySQL depuis le réseau (par le socket MySQL ; par contre on accède à distance par SSH et ensuite on se connecte à SQL une fois rentré dans la machine donc en « local ». • empêcher les connexions anonymes • supprimer la base de test (accessible par défaut par tous, même anonymes) et les privilèges qui autorisent tout le monde à accéder à toutes les bases commençant par test_

### Commandes MySQL (base)

##### Se connecter 

```bash
mysql -h 'host '-u 'user' -p 
```

mysql -h localhost -u root -p

- - -h premet de préciser l'hôte (host)
  - -u permet de préciser l'utilisateur (user)
  - -p indique que nous allons ensuite taper le mot de passe (password)

##### Lister les bases

```mysql
SHOW DATABASES;
```

##### Selectionner une base
```mysql
USE "nom de la bdd";
```

##### Lister les tables d'une base

- si base selectionné

  ```mysql 
  SHOW TABLES;
  ```
- sinon

  ```mysql 
  SHOW TABLES FROM "nom de la bdd";
  ```
##### Créer une base 

```mysql
CREATE DATABASE "nom bdd";
```

##### Créer un utilisateur 

```mysql
CREATE USER 'user'@'localhost';
```

##### Affecter une mot de passe à un utilisateur

```mysql
SET PASSWORD FOR 'user' = PASSWORD('mot de passe');
```

##### Donner tout les droits à un utilisateur sur un BD (voir "Les privilèges" pour les types de droits)

```mysql
GRANT ALL PRIVILEGES ON 'BASE.*' TO user@localhost IDENTIFIED by 'mot de passe';
```

##### Revoquer les droits d'un utilisateur

```mysql
REVOKE ALL PRIVILEGES ON 'BASE.*' FROM user;
```

##### Supprimer un compte utilisateur

```mysql
DROP user 'test'@'localhost';
```

### Les privilèges (droits)

> En matière de bases de données, on parle de ***privilèges*** pour parler des *droits*. Ces privilèges s'appliquent à des comptes. Tel compte à le droit de faire telle action.

 Les information des utilisateurs et des privilèges sont stockées dans la base de données mysql :

- table user ➡️ utilisateurs(privilèges globaux) 

**4 tables pour stocker les privilèges users : **

- **db** : privilèges au niveau des bases de données. 
- **tables_priv** : privilèges au niveau des tables. 
- **columns_priv** : privilèges au niveau des colonnes. 
- **procs_priv** : privilèges au niveau des routines (procédures et fonctions stockées).

#### Droits principaux

- **GRANT** : le droit de donner des droits
- **CREATE** : Créer des utilisateurs, des bases, des tables ou des index
- **DROP** : Supprimer des utilisateurs, des bases, des tables ou des index
- **ALTER** : modifier la structure de tables
- **DELETE** : Supprimer des données
- **INSERT** : Ajouter des données
- **UPDATE** : Modifier des données

##### exemple (tp mysql) :

• Ajouter le droit de faire un INSERT sur la table « table1 » pour l’utilisateur “invite” : 

```mysql
GRANT INSERT ON table1 TO invite;
```

• Attribuer tous les droits à l’utilisateur “invite” sur la base de données utilisée : 

```mysql
GRANT ALL ON * TO invite;
```

• Retirer le droit de faire un INSERT sur toutes les tables de la BDD utilisée pour l’utilisateur « invite » :

```mysql
REVOKE INSERT ON * FROM invite;
```

• Retirer le droit de faire un INSERT sur la table « table1 » pour l’utilisateur “invite” : 

```mysql
REVOKE INSERT ON table1 FROM invite;
```

• Retirer tous les droits à l’utilisateur “invite” sur toutes les tables de la BDD utilisée : 

```mysql
REVOKE ALL PRIVILEGES ON * FROM invite
```

> :warning: - ne jamais donner **WITH GRANT OPTION** à un utilisateur non administrateur système 

### Supervision et "tuning" MySQL

##### **MySQLTuner** :

> Script **PERL** qui permet d'optimiser les performances d'un serveur de bases de données MySQL en faisant un diagnostic, voir: https://github.com/major/MySQLTuner-perl

##### **Mytop** 

> Outil "top like" écrit en PERL mais pour les base de données. (intégré à deb9)

- utilisation :

  ```bash
  mytop --prompt -d "base de donnée"
  ```

Si cela ne fonctionne pas , installer les dépendances PERL suivante :

```bash
 apt-get install libconfig-inifiles-perl
```

**Mysql only :**

fichier de confg d'exemple dans /usr/share /doc/mysql-server-5.X.XX

##### **Tuning :** 

1. **key_buffer** 

   ```mysql
   SHOW GLOBAL STATUS
   ```

   ligne "**key_reads/key_read_requests**" radio < 0.01 pour éviter accès disque 

   sinon augmenter le key_buffer

2. **thread_concurrency** (machine multicoeur)

   x2 nom de coeurs

3. **Activer le log des requêtes lentes** 

   slow_query_log=1 

   slow_query_log_file=/var/log/mysql/mysql-slow.log 

   - *Et demander d'indiquer le temps le plus lent :* 

   long_query_time = 2 

   - *Analyser les résultats:*

   ```bash
   tail -n 1000 /var/log/mysql/mysql-slow.log 
   ```

4. **check "connection leakage"**

   ```mysql
   SHOW PROCESSLIST;
   ```

5. **Analyser les verrous :**

   ```mysql
   SHOW STATUS LIKE 'Table%';
   SHOW ENGINE INNODB STATUS (pour innodb)
   ```

### Sauvegarde et restauration

#### backup à chaud

- Utilisation de "**mysqldump**" exemples :

  - bases précises :

    ```bash
     mysqldump --user=root -p --databases mysql wiki > wiki_bd_backup.sql
    ```

  - all base :

    ```bash
    mysqldump --user=root -p --all-databases | gzip > save.mysql.sql.gz
    ```

- Possibilité d'exporter via outils graphiques (phpmyadmin ou autre)

#### backup à froid : 

> Elle consiste à sauvegarder les fichiers “physiques” du serveur de données. Elle nécessite, par définition l’arrêt du serveur. 
>
> Aucune activité n’est donc possible. C’est un avantage pour le DBA mais il est parfois difficile voire impossible de stopper l’activité en cours. En revanche si l’on dispose d’un réplica on peut tout à fait l’envisager. Celui ci se “re synchronisera” à son redémarrage. La sauvegarde à froid consiste à archiver certains fichiers du serveur. **/var/lib/mysql/** et **/var/log/mysql.log** 
>
> Pour les tables MyISAM il s’agit plus précisement des fichiers **.FRM, .MYI et .MYD** Pour les tables InnoDB : **.FRM**, les fichiers de données (ibdata), les fichiers des tables .ibd (si vous êtes en innodb_file_per_table) et les fichiers journaux (iblogfile) intermédiaires.

#### Restauration

dump en selectionnant la base :

```mysql
use database;
source /chemin/du/fichier/backup.sql
```

ou pour le scripting : 

```bash
mysql -u myuser -p < /chemin/du/fichier/backup.sql ( automatisation totale avec --password = < mot de passe >
```

> pour restaurer à froid, il suffit de couper le service et de déposer les fichiers aux bons emplacements

#### Automatisation des sauvegardes

##### automysqlbackup, Script "tout en un" :

- Notification par mel 
- Compression et chiffrement des sauvegardes
- Rotation des sauvegardes configurable 
- Sauvegardes incrémentales

```bash
apt-get install automysqlbackup
```

à configure dans  **/etc/default/automysqlbackup**

##### rsnapshot : 

```bash
apt-get install rsnapshot
```

voir https://wiki.debian-fr.xyz/Rsnapshot

##### outil backup à chaud : 

https://www.percona.com/doc/percona-xtrabackup/LATEST/index.html

### Migration Mysql vers MariaDB (deb<9)

- Maj du serveur Debian

  ```bash
  apt-get update && apt-get upgrade
  apt-get install python-software-properties software-properties-common
  ```

- Ajout des dépot MariaDB voir :  https://downloads.mariadb.org/mariadb/repositories/

- Si Mysql > 5.5 migration directement sur 10.1 sinon 5.5

- update des paquets et installe :

  ```bash
  apt-get update && apt-get install mariadb-server
  ```



### Cluser MariaDB/Galera

> Cluster synchro en maître-maître
>
> En production favoriser 3 serveurs minimum

- Création du cluster :

  ```bash
  service mysql stop
  mysqld --wsrep-new-cluster
  or
  galera_new_cluster
  ```

- sur les autres noeuds :

  ```bash
  service mysql stop
  mysqld --wsrep_cluster_address=gcomm://ip(ou nom DNS du serveur qui a créé le cluster)
  ```

- voir le statut de la syncrho :

  ```mysql
  SHOW STATUS LIKE 'wsrep_%'
  ```

  configurer affiné voir http://www.severalnines.com/New-Galera-Configurator/index.html

### Troubleshoot

#### Table corrompue

- Message d'erreur 

  > “Table is marked as crashed and should be repaired” 

- Solution : 

  ```mysql
  repair table bd.tablename;
  ```


#### Récupération de mot de passe root MySQL

arrêt du service et redémarrage en mode rescue

```bash 
systemctl stop mysql && mysqld_safe --skip-grant-tables & 
```

Connexion en root 

```bash
mysql -u root
```
dans la console mysql :
```mysql
#selectionner la base mysql
mysql> use mysql; 
#changement du password de l'()
mysql> update user set password=PASSWORD("nouveaumotdepasse") where user='root'; 
#on sauvegarde
mysql> flush privileges; 
#et on quitte
mysql> quit 
```

arrêt du mode rescue

```bash
systemctl stop mysql
```

test de connexion :

```bash
mysql -u root -p
```

## PostgreSQL

:information_source: pour l'installation chercher LAMP

### Commande 

- Modifier le mot de passe de l'utilisateur système postgres 

  ```BASH
  passwd postgres
  ```

- Modifier le mot de passe de l'utilisateur administrateur de la base postgres 

  ```bash
  su - postgres
  psql -d template1 -c "ALTER USER postgres WITH PASSWORD 'mot de passe'"
  ```

  ou

  ```bash
  psql
  postgres=# \password postgres
  ```

- Créer une base 

  ```bash
  su - postgres
  createdb "nom de la base"
  ```

- Se connecter à une base

  ```bash
  psql "nom de la base"
  ```

- help

  ```plsql
  \h
  ```

- fermer la connexion

  ```psql
  \q
  ```

- Créer un utilisateur 

  ```bash
  psql -d template1 -c "create user "nom de l'user" with password 'motdepasse'"
  ```

- Pour gérer la base à distance

  ```bash
  su - postgres
  psql template1 < /usr/share/postgresql/9.6/extension/adminpack--1.0.sql
  exit
  ```

  ou

  ```bash
  postgres=# CREATE EXTENSION adminpack;
  CREATE EXTENSION
  ```

- Activation des connexion TCP/IP 

  ```bash
  vim /etc/postgresql/9.6/main/postgresql.conf
  	listen_addresses = 'localhost'
  	password_encryption = on

  service postgresql restart
  ```

### Administration web (voir section LAMP pour l'installation) :

```bash
apt-get install phppgadmin
```

> modifier /etc/apache2/conf-available/phppgadmin.conf pour les accès

```bash
ln /etc/apache2/conf-available/phppgadmin.conf /etc/apache2/sites-available/.
a2ensite phppgadmin.conf
```

- restart des services  apache2 et postresql et connexion au site http://ipduserv/phppgadmin

### Réplication "streaming" PostgresSQL

##### Prérequis

2 serveurs avec PostgreSQL (maître et esclave

- ##### Serveur maître :

  ```bash
  vim /etc/postgresql/9.6/main/postgresql.conf     
  	listen_addresse = '*'
  	wal_level = hot_standby wal_keep_segments = 10
  	max_wal_senders = 3
  ```

- User pour la réplication

  ```bash
  psql -h localhost -U postgres -W -c CREATE USER utilrepl WITH REPLICATION PASSWORD 'motdepasse';"
  ```

- Droit de connexion pour l'user de réplication

  ```bash
  vim /etc/postgresql/9.6/main/pg_hba.conf 
  host    replication     utilrepl        ip esclave/cidr          md5
  ```

- On arrête de le service

  ```bash
  service postgresql stop
  ```

- ##### Server escalve :

  ```bash
  vim /etc/postgresql/9.6/main/postgresql.conf
  listen_addresses = '*' 
  hot_standby = on
  ```

- On arrête l'esclave et on détruit tous les fichiers de la base

  ```bash
  /etc/init.d/postgresql stop 
  cd /var/lib/postgresql/9.6/main/ 
  rm -rf * 
  vim /var/lib/postgresql/9.6/main/recovery.conf 
  primary_conninfo = 'host=serveur maître port=5432 user=utilrepl password=motdepasse' 
  standby_mode = on
  ```

- On copie tout du maître vers l'esclave

  ```bash
  rsync -av /var/lib/postgresql/9.6/main/* ip esclave:/var/lib/postgresql/9.6/main/
  ```

- Redémarrer les services sur les deux serveurs

  ```bash
  service postgresql start
  ```

- check de la réplication

  ```bash
  psql -h localhost -U postgres -W -c "select * from pg_stat_replication;"
  ```


### Troubleshoot

#### Récupération de mot de passe oublié 

- Arrêt du service

  ```bash
  service postgresql stop
  ```

- modification de pg_hba.conf

  ```bash
  vim /usr/lib/pgsql/data/pg_hba.conf
  	local all all ...
  #modifier en : 
  	local all postgres trust
  ```

- Relancer le service et changer le mdp 

  ```bash
  service postgresql start 
  su - postgres $ psql -d template1 -U postgres 
  alter user postgres with password 'votrenouveaumotdepasse';
  #version une commande
   psql -U postgres template1 -c "alter user postgres with password 'votrenouveaumotdepasse';"

  ```

- Remodifier pg_hba.conf pour avoir "local all postgres ident" et relancer le service !

### Tuning

> voir https://www.dsfc.net/logiciel-libre/postgresql/tuning-postgresql-configuration-memoire-linux/

### Backup

> Voir section mysql sinon : 
>
> http://www.pgbarman.org/