# Deploiement d'un projet django sur le web
Documentation permettant de déployer un projet django sur un serveur Ubuntu avec apache2 et mod-wsgi. Hebergeur utilisé : Hostinger  

## Préparer le projet DJANGO pour la mise en prod
On n'expliquera pas ici comment créer un projet Django. Si besoin utiliser le projet de base proposé dans les tutos de djangoproject 
https://docs.djangoproject.com/en/4.2/intro/tutorial01/

On utilisera git et un repo par exemple sur github [https://github.com/monprojet.git]

Modification à apporter dans le fichier setting.py de django :

DEBUG = os.environ.get('DEBUG', True)
SECRET_KEY = os.environ.get('SECRET_KEY', 'django-insecure-...')
ALLOWED_HOSTS = [[adresse_ip_ssh], [nom_de_domaine]]


## Choisir et configurer son hebergeur
Choisir un hebergement permettant d'avoir un accès ssh et des accès root.
Dans l'exemple si dessous on utilise hostinger.com

Sur hostinger.com
- Souscrire à un plan VPS (le plan hébérgement ne donne pas l'accès root)
- Créer un mot de passe root puis noter l'adresse IP de notre serveur [adresse_ip_ssh]
- Pour plus de sécurité ajouter une clé SSH 

## Configurer son serveur 
### Accès ssh
ssh root@[adresse_ip_ssh]

### Environnement
sudo apt-get install git python3-pip apache2 libapache2-mod-wsgi-py3 python3.10-venv

Par default apache2 rend public le fichier /var/www/html/index.html, on peut tester sur un navigateur [adresse_ip_ssh] et voir l'affichage de la page par default

### Récuperer notre projet
cd /var/www/
git clone [https://github.com/monprojet.git]

### Builder le project django 
python -m venv .env
source .env/bin/activate
pip install -r requirements.txt
python manage.py collectstatic
python manage.py migrate
python manage.py makemigrations

### Configuration d'apache 
La configuration d'apache se fait dans le dossier /etc/apache2/sites-available/
cd /etc/apache2/sites-available/

Dans le dossier sites-available de ce repo on trouve 3 configurations en exemple
projet.conf, configuration pour le site http
projet-ssl.conf, configuration pour le site https
projet-redirect.conf, redirection du http vers https

projet.conf et projet-redirect.conf ne peuvent pas être utilisé en même temps. projet.conf est donné pour l'exemple mais enfaite il ne sera pas utilisé à terme car on souhaite avoir notre site en https tout le temps.

cmd
a2ensite  
a2dissite
systemctl restart apache2
systemctl status apache2
a2enmod ssl
a2dismod ssl
apache2ctl -S

Afficher les sites actifs 
a2query -s

Afficher les sites dispo

#### pour un site en http
a2dismod ssl
a2ensite projet.conf
systemctl restart apache2

- http://[adresse_ip_ssh] donne accès au site
- https://[adresse_ip_ssh] ne renvoi rien 

 #### pour un site en https seulement 

a2enmod ssl
a2dissite projet.conf
a2ensite projet-ssl.conf
a2ensite projet-redirect.conf
systemctl restart apache2

- http://[adresse_ip_ssh] redirige sur https://
- https://[adresse_ip_ssh] ne renvoi rien 
