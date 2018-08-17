## PostgreSQL

ℹ️ pour l'installation chercher LAMP

### Commande

- Modifier le mot de passe de l'utilisateur système postgres 

  ```bash
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