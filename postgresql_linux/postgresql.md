# Créer un base de donnée POSTGRSQL SOUS Linux

## Installer postgresql sous linux
```
sudo apt-get install postgresql
```

## Ouvrir l'invit de commande postgresql

#### *Se connecter en tant que postgres*
```
sudo -i -u postgres 
```
  
  
#### *Créer une database*
```
createdb mydb
```
#### *Rentrer dans postgreSQL*
```
psql
```
#### *Rentrer dans la database mydb*
```
psql mydb
```
#### *Commande psql*
```
\h                      # help
\dt                     # Lister les tables
SELECT * FROM TABLE     # Commande SQL  
```


# Créer un utilisateur
```
CREATE USER user_name WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE mydb TO user_name
```

A partir de la notre base de donnée `mydb` existe, par default elle n'est accessible qu'en local. On peut tester avec 
```
psql -h localhost -U user_name -d mydb
```

Si besoin 
```
sudo lsof -i -P -n | grep LISTEN
systemctl status postgresql
systemctl stop postgresql
systemctl start postgresql
```

## Ouvrir le port postgreSQL
Pour ouvrir le port 5432 (port par default utilisé par postgrSQL) pour que votre base de données PostgreSQL soit accessible de l'extérieur, vous devez suivre les étapes suivantes :

1. **Modifier le fichier de configuration PostgreSQL** :  
Tout d'abord, vous devez vous assurer que PostgreSQL accepte les connexions distantes. Pour cela, vous devez modifier le fichier postgresql.conf, qui est généralement situé dans le dossier /etc/postgresql/14/main :

```
sudo nano /etc/postgresql/14/main/postgresql.conf
````
Recherchez la ligne contenant listen_addresses et changez sa valeur à '*':

```
listen_addresses = '*'
```
Cela permettra à PostgreSQL d'accepter les connexions de n'importe quelle adresse IP.
  
  
2. **Modifier le fichier pg_hba.conf** :  
Vous devez également autoriser les connexions à votre base de données à partir de l'adresse IP de votre client. Pour cela, vous devez modifier le fichier pg_hba.conf, qui est généralement situé dans le même dossier que postgresql.conf :

```
sudo nano /etc/postgresql/14/main/pg_hba.conf
```
À la fin de ce fichier, ajoutez une ligne comme celle-ci :

```
host    all             all             IP_CLIENT/32         md5
```
Remplacez `IP_CLIENT` par l'adresse IP de votre client. Cette ligne permet à toute connexion provenant de cette adresse IP d'accéder à toutes les bases de données et à tous les utilisateurs si le mot de passe est correct.

3/ **Redémarrer PostgreSQL** :  
Après avoir effectué ces modifications, vous devez redémarrer le service PostgreSQL pour que les modifications prennent effet :

```
sudo service postgresql restart
````

4. **Configurer le pare-feu** :  
Si vous utilisez UFW comme gestionnaire de pare-feu, vous pouvez ouvrir le port 5432 avec la commande suivante :

```
sudo ufw allow 5432
```
Si vous utilisez un autre gestionnaire de pare-feu, la commande peut être différente.