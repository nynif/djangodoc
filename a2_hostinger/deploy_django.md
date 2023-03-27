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

