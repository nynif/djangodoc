# Deploiement d'un projet django sur le web
Documentation permettant de déployer un projet django sur un serveur Ubuntu avec apache2 et mod-wsgi. Hebergeur utilisé : Hostinger  

## Préparer le projet DJANGO pour la mise en prod
On n'expliquera pas ici comment créer un projet Django. Si besoin utiliser le projet de base proposé dans les tutos de djangoproject 
https://docs.djangoproject.com/en/4.2/intro/tutorial01/

On utilisera git et un repo par exemple sur github `https://github.com/monprojet.git`

Modification à apporter dans le fichier `setting.py` de django :

```
DEBUG = os.environ.get('DEBUG', True)
SECRET_KEY = os.environ.get('SECRET_KEY', 'django-insecure-...')
ALLOWED_HOSTS = [[adresse_ip_ssh], [nom_de_domaine]]
```
*(Voir plus loin pour l'utilisation de variable d'environnement)*

## Choisir et configurer son hebergeur
Choisir un hebergement permettant d'avoir un accès ssh et des accès root.  
Dans l'exemple si dessous on utilise [hostinger](https://www.hostinger.fr)

- Souscrire à un plan VPS (le plan hébérgement ne donne pas l'accès root)
- Créer un mot de passe root puis noter l'adresse IP de notre serveur `[adresse_ip_ssh]`
- Pour plus de sécurité ajouter une clé SSH 

## Configurer son serveur 
### Accès ssh
ssh root@[adresse_ip_ssh]

### Environnement
```
sudo apt-get install git python3-pip apache2 libapache2-mod-wsgi-py3 python3.10-venv
```

Par default apache2 rend public le fichier `/var/www/html/index.html`, on peut tester sur un navigateur `[adresse_ip_ssh]` et voir l'affichage de la page par default

### Récuperer notre projet
```
cd /var/www/
git clone [https://github.com/monprojet.git]
```

### Builder le project django 
```
python -m venv .env
source .env/bin/activate
pip install -r requirements.txt
python manage.py collectstatic
python manage.py migrate
python manage.py makemigrations
```

### Configuration d'apache 
La configuration d'apache se fait dans le dossier `/etc/apache2/sites-available/`
```
cd /etc/apache2/sites-available/
```

Dans le dossier `sites-available` de ce repo on trouve des configurations en exemple
- projet.conf, configuration pour un site http
- projet-ssl.conf, configuration pour un site https
- projet-redirect.conf, configuration pour une redirection du http vers https
- alias.conf, configuration pour donner accès direct à une ressource (exemple robot.txt) 
- site-index.conf, configuration pour un site simple index.html
- flask.conf, configuration pour un serveur flask

**SI L'ON VEUT METTRE PLUSIEURS SITE SUR UN MÊME SERVEUR**
- Utiliser plusieurs fichier conf
- Chacun aura <VirtualHost *:80>, tous sur le port 80
- Le ServerName sera différent et cela suffit

### Commande principal d'apache
```
a2ensite  
a2dissite
systemctl restart apache2
systemctl status apache2
a2enmod ssl
a2dismod ssl
apache2ctl -S
a2query -s
```

#### pour un site en http
```
a2dismod ssl
a2ensite projet.conf
systemctl restart apache2
```
> http://[adresse_ip_ssh] donne accès au site   
https://[adresse_ip_ssh] ne renvoi rien 


#### pour un site en https seulement 
```
a2enmod ssl
a2dissite projet.conf
a2ensite projet-ssl.conf
a2ensite projet-redirect.conf
systemctl restart apache2
```

>http://[adresse_ip_ssh] redirige sur https://  
https://[adresse_ip_ssh] ne renvoi rien 

### Reglage du DNS
Le DNS sert à diriger notre `[nom_de_domaine]` à `[adresse_ip_ssh]`
Pour se faire on se rend sur la platforme qui gère notre nom de domaine, dans l'exemple hostinger, dans l'onglet enregistrements DNS du nom de domaine qui nous interesse.
Il nous faut un et un seul enregistrements de type A, s'il existe on l'edite, sinon on le crée.
```
Type A
Nom @
Contenu [adresse_ip_ssh]
TTL 1800
```
Attendre quelques minutes la propagation

> https://[nom_de_domaine] donne alors accès au site

Pour tester sur la propagation c'est faite on peut utiliser [toolbox](https://toolbox.googleapps.com/apps/dig/#CAA/)

#### Cas de l'utilisation de sous domaine
Si on veut utiliser un sous domaine on peut alors avoir plusieurs enregistrement de type A, on indiquera le nom de domaine dans la valeur Nom (à la place du @ précedent)

### Reglage du SSL
Le site est accessible en HTTPS mais si on garde les parametres d'origine on utilise des certificats qui ne sont pas vraiment sécurisé. Il faut donc créer et authentifier un certificat.   
Dans l'exemple on utilise [zerossl.com](zerossl.com) pour generer un certificat SSL.

On obtient alors 2 fichier:  
- certificat.crt a placer sur le serveur dans /etc/ssl/certs/
- private.key /etc/ssl/private/

pour se faire on peut utiliser la commande scp 
```
scp -r -p certificat.crt root@[adresse_ip_ssh]:/etc/ssl/certs/certificat.crt
scp -r -p private.key root@[adresse_ip_ssh]:/etc/ssl/certs/private.key
```

Dans la config d'apache (/etc/apache2/sites-available/projet-ssl.conf) on modifie également les chemins de 
```
SSLCertificateFile      [/etc/ssl/certs/certificat.crt]
SSLCertificateKeyFile   [/etc/ssl/private/private.key]
```

Doc
- https://www.it-connect.fr/configurer-le-ssl-avec-apache-2%EF%BB%BF/?utm_content=cmp-true


### Variable d'environnement
Côté serveur on aura besoin d'utiliser des variables d'environnements pour le fichier ‰`setting.py`  
Pour se faire on utilise la librairie https://django-environ.readthedocs.io  

  
Dans le projet django setting.py il faudra donc ajouter
```
import environ
env = environ.Env()
environ.Env.read_env(os.path.join(BASE_DIR, '.envfile'))

SECRET_KEY = env.str('SECRET_KEY', default='django-insecure-s&yw&ov_$9)pyd(r(dfdsfgdsfdsfdsfb6r48&344=')
DEBUG = env.bool('DEBUG', default=False)
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS', default=['127.0.0.1'])
```

et on crée un fichier `.envfile` à la racine de notre projet 
ce fichier sera gitignore donc il faudra le créer manuellement côté serveur
```
DEBUG=False
SECRET_KEY='my_secret_key'
ALLOWED_HOSTS=[adresse_ip_ssh], [nom_de_domaine]
```
ATTENTION pour les list comme ALLOWED_HOSTS, ne pas mettre de guillemet

Pour générer une secret key
```
python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
```

### Procèdure pour mettre à jour
```
ssh root@[adresse_ip_ssh]
cd /var/www/
git fetch
git pull 
systemctl restart apache2
```
Si besoin 
```
source .env/bin/activate
pip -r install requirements.txt  
python manage.py collectstatic
python manage.py migrate
python manage.py makemigrations
systemctl restart apache2
```


### Procedure pour tester ce projet sur un vps
sur le serveur via ssh
```
git clone 
cd /etc/apache2/sites-available/
touch monprof.conf
```

## Erreur possible

### Django 
Error de permission sur le dossier static
chmod 777 static
(+ systemctl restart apache2)

## Autre commandes utiles
```
python manage.py check --deploy
systemctl restart apache2
```