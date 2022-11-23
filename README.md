# Playbook Ansible pour Decidim sur Debian 11 / Ubuntu 20


## Pré-requis
- Sur le serveur à provisionner (Decidim) :
    - Créer un utilisateur dédié à la gestion de la stack Decidim
    - Ajouter cet utilisateur au groupe sudoers
- Sur votre serveur Ansible
    - Cloner le dépot contenant le role Ansible pour le provisionnement de Decidim
    - Ajouter le serveur à provisionner dans votre inventaire Ansible (**inventory.yml**) en remplissant les variables suivantes avant d'exécuter le playbook
        ```
        decidim-server:
            ansible_host:
            git_organization:
            git_root:
            git_repository:
            repository_branch:
            decidim_host:
            certbot_email_address:
            database_name:
            secret_key_base:
            decidim_deployment_path: 
            ssl_certificate_path: 
            ssl_certificate_key_path: 
            decidim_admin_email: 
        ```

## Variables
*Se référer au fichier **inventory-example.yml** pour les exemples*

- **ansible_host** : Adresse IP du serveur à provisionner
- **git_organization** : Le nom de l'organisation Git (exemple : `OpenSourcePolitics`)
- **git_root** : URL racine du serveur Git (exemple : https://github.com)
- **git_repository** : Nom du dépot Decidim hébergé sur Git (exemple : `decidim-app`)
- **repository_branch** : Branche git du dépot à déployer (par défaut : `master`)
- **decidim_host** : sous-domaine d'accès à la plateforme Decidim (exemple : `decidim-deb.osp.dev`)
- **database_name** : Nom de la base de donnée Decidim (par défaut : `osp_app`)
- **secret_key_base:** : La secret key base utilisée avec l'archive de données PostgreSQL importée
- **decidim_deployment_path** : Chemin d'installation pour l'environnement Ruby on Rails et la stack Decidim
- **ssl_certificate_path** : Chemin vers le certificat SSL pour la configuration Nginx
- **ssl_certificate_key_path** : Chemin vers la clé du certificat SSL pour la configuration Nginx
- **decidim_admin_email** : addresse email d'un administrateur de la plateforme, **il doit exister sur chacune des organisations existantes** 

## Lancer le playbook d'installation

Une fois les pré-requis en place, lancer la commande suivante depuis de le répertoire Ansible pour lancer le déploiement du role Ansible sur le serveur à provisionner
```
ansible-playbook -u decidim --private-key="~/path/to/ssh-key" -i inventory.yml playbook.yml
```

## Résultat attendu

Apres exécution du playbook sur le serveur cible, celui-ci devrait avoir :

- Installer les dépendances liées à Ruby on Rails et Decidim
- Installer le service Memcached
- Installer Rbenv, gestionnaire de versions Ruby
- Installer le dépot de code de Ruby
- Installer la derniere version stable de NodeJS
- Installer la derniere version stable de Yarn
- Installer et initialiser PostgreSQL
- Compléter l'ensemble des étapes liées au bon fonctionnement de Decidim, à savoir : 
    - Téléchargement du dépot de code dans le répertoire créé à partir de la variable d'inventaire **git_repository**
    - Installation de la version 2.7.1 de Ruby
    - Installation de la version 2.2.29 de Bundler
    - Installation des gems liées à Decidim (présentes dans le fichier **Gemfile**)
    - Création d'un fichier **.env** contenant les variables d'environnement nécéssaires au bon fonctionnement de l'application et renseignées en amont dans le fichier **inventory.yml**
    - Installation des migrations Rails
    - Précompilation des assets statiques de l'application (CSS et JS)
- Configurer un logrotate pour les logs génerés par Decidim
- Installer Nginx et configurer le sous-domaine d'accès pour Decidim (variable d'inventaire **decidim_host**)
- Installer et configurer Phusion Passenger pour l'application Decidim
- Configurer Sidekiq pour le traitement des taches asynchrones executées par l'application Decidim
- Installer Redis
- Installer Certbot avec création d'un certificat SSL Let's Encrypt pour le sous-domaine d'accès pour Decidim (variable d'inventaire **decidim_host**)

## Finalisation

Extraire l'archive des fichiers uploadés dans le dossier de l'application Decidim qui correspond à 
```
/home/{{ ansible_user }}/{{ git_repository }}/public/uploads
```

exemple : 
```
tar -jxvf /path/to/archive /home/decidim/decidim-app/public/uploads
```
