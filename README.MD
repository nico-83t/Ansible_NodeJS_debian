# Ansible configuration for NodeJS APPs
## Config et deploy en vidéo
[![YoutubeVidéo](https://i.ytimg.com/vi/kruXJx68oHA/hqdefault.jpg)](https://youtu.be/kruXJx68oHA)
## Liste des taches
### Starter
[Installe une liste de packet](roles/starter/tasks/main.yml#L5)
- git
- vim
- curl
- openssl
- build-essential
- libssl-dev

### Deployer
[Utilisateur et structure](roles/deployer/tasks/main.yml)
- Création d'un nouvel utilisateur (username dans [playbook.yml](playbook.yml#L6))
- Configuration des accès ssh
- Création des dossiers de déploiement par domaine et par environnement 
`/home/{{ username }}/www/{{ domain }}/{{ env }}`
`/home/deploy/www/mydomain.fr/production`
- Création des dossier pour les logs
`/home/deploy/www/mydomain.fr/logs`

### Node
[Installe les dépendances pour le fonctionnement de node](roles/node/tasks/main.yml)
- Installation de nvm (version dans [playbook.yml](playbook.yml#L11))
- Installation de node (version dans [playbook.yml](playbook.yml#L10))
- Installation de pm2

### Mysql
[Installe mysql](roles/mysql/tasks/main.yml)
- Installation des dépendances
- Suppression des utilisteurs anonyme
- Création d'une base de donnée (nom dans [playbook.yml](playbook.yml#L8))
- Attribution des droits à l'utilisateur (username dans [playbook.yml](playbook.yml#L6))

### Nginx
Attention, il faut avoir déployé l'application avant cette étape !
[Installe et configure nginx](roles/nginx/tasks/main.yml)
- Installation de nginx
- Configuration des apps par domaine et par environnement ([ici](roles/nginx/tasks/main.yml#L13) et [ici](roles/nginx/tasks/main.yml#L23))
- Suppression de la config par  défaut
- Redemarage de nginx

## Configuration
#### Server side
- Sur serveur Debian ou Ubuntu (testé sur Debian Stretch)
- Il est nécessaire d'avoir pré-configuré un accés root ssh avec les clefs associées
```shell
ssh root@xxx.xxx.xxx.xxx

# Connexion successfull !
```

#### Client side
hosts file
```
# Modifier l'adresse ip dans le fichier host avec celle de votre serveur
# Il est possible d'utiliser plusieurs adresses ip simultanement (1 ligne / ip)
[web]
71.85.130.169
```

#### Playbook.yml
Le fichier `playbook.yml` contient un esemble de variables dans les premières lignes, veuillez les modifier pour les adapter à votre projet.
```yml
vars: 
  user: deploy # nom d'utilisateur qui sera utilisé pour l'accés ssh et la création des dossiers
  domain: mydomain.fr # nom de domaine utilisé pour la configuration nginx et la création des dossiers
  db_name: my_db_name_production # nom de la base de donnée qui va être créer le user défini plus haut aura tout les droits sur cette base
  my_db_password: "my_db_password" # mot de passe mysql qui sera attribué à l'user défini plus haut
  node_version: "8.11.3" # version de node à installer via nvm
```

##### Lancer le script ansible
playbook.yml
- starter => Installe les packages de base necessaires
- deployer => Ajoute et configure un acces ssh pour l'user "deploy"
- node => Installe nvm (Aller à la section node pour finir la configuration)
- mysql => Installe un serveur mysql, créer une base de données, un user, et attribue les droits de la base de données à l'user 
- DEPLOY de l'app à l'aide de capistrano (voir Capistrano Deploy)
- nginx => Installe et configure nginx pour un ou plusieurs environnements (avant d'executer les tasks nginx il faut avoir deploy et démarré l'app sur le serveur via pm2, puis il faut modifier le fichier `nginx/tasks/main.yml` avec les bonnes valeurs)
```shell
ansible-playbook -i hosts playbook.yml
```

## Capistrano Deploy
[Capistrano](https://capistranorb.com/) est un outil écrit en ruby mais est compatible avec tout les languages. Sont rôle va être de cloner un répertoire GitHub vers un serveur et de créer une architecture de dossier permettant le deploiement continue.

Capistrano à besoin de ruby pour fonctionner, je vous conseille d'installer la version `2.5.0` à l'aide du gestionnaire de version [rbenv](https://github.com/rbenv/rbenv)
```shell
rbenv install 2.5.0
rbenv global 2.5.0

ruby -v
ruby 2.5.0p0 (2017-12-25 revision 61468) [x86_64-darwin17]
```
Rendez vous maintenant à la racine de votre projet web
```shell
touch GEMFILE
```
Puis ajouter capistrano au GEMFILE
```ruby
source 'https://rubygems.org'

group :development do
  gem "capistrano", "~> 3.11", require: false
end
```
Puis installer capistrano avec
```shell
bundle install

# It return
# Fetching gem metadata from https://rubygems.org/..........
# Resolving dependencies...
# Using rake 12.3.1
# Fetching net-ssh 5.0.2
# Installing net-ssh 5.0.2
# Fetching net-scp 1.2.1
# Installing net-scp 1.2.1
# Fetching sshkit 1.18.0
# Installing sshkit 1.18.0
# Fetching airbrussh 1.3.1
# Installing airbrussh 1.3.1
# Using bundler 1.16.3
# Fetching concurrent-ruby 1.1.3
# Installing concurrent-ruby 1.1.3
# Using i18n 1.1.1
# Fetching capistrano 3.11.0
# Installing capistrano 3.11.0
# Bundle complete! 1 Gemfile dependency, 9 gems now installed.
# Use `bundle info [gemname]` to see where a bundled gem is installed.

bundle exec cap install
```
A la suite de la dernnière commande, une liste de fichiers et dossiers vont être créé
```shell
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb # Configuration du serveur et de l'user qui se connecte
│   │   └── staging.rb
│   └── deploy.rb # Configuration partagé
└── lib
    └── capistrano
            └── tasks
```
Nous allons modifier uniquement `deploy.rb` et `production.rb`
deploy.rb
```ruby
set :application, "my_app_name" # Nom de l'app
set :repo_url, "git@github.com:USERNAME/my_app_name.git" # Repos Github du projet
set :deploy_to, "/home/deploy/www/mydomain.fr/production" # Repertoire de deploy
append :linked_dirs, "uploads", "logs", ... # Dossier liés (dossier qui resteront les mêmes entre chaques deploy)
set :keep_releases, 5 # Nombre de copie gardé côté serveur (au bout de 5 la plus ancienne est écrasé)
```
production.rb
```ruby
server "xxx.xxx.xxx.xxx", user: "deploy", roles: %w{app db web} # Adresse ip du serveur et user de deploiement
```
Une fois que la configuration de capistrano est terminé, il suffit de run
```shell
cap production deploy
```

Capistrano va s'occuper de déployer le contenu du repos vers le repertoire du serveur indiqué dans la configuration.
Attention capistrano utilise beaucoup de symlink, l'ensemble des déploiements se retrouve dans le repertoire `releases` capistrano créer un répertoire `current` dans `/home/deploy/www/mydomain.fr/production/` qui est en fait un symlink de la dernière release, il faut donc configurer nginx pour pointer vers le repertoire `current`

Les dossiers `uploads` et `logs` configuré dans les `linked_dir` vont eux aussi être des symlink à l'interieur du dossier `current` qui vont en réalité pointer vers `/home/deploy/www/mydomain.fr/production/shared` ce qui permet de conserver ces derniers entre chaque deploiement.

Une fois le deploiement terminé :
- Se connecter en ssh au serveur
- Se rendre dans le repertoire de deploy `/home/deploy/www/mydomain.fr/production/current`
- Run :
```shell
npm install

pm2 start bin/www
```
Cette dernière commande va permettre de lancer l'app Node.JS (sur le port configuré dans `bin/www`)

Executer ensuite la dernière task du Ansible playbook (commenter les premières déjà executer et dé-commenter la task `nginx`)

***Rendez vous sur l'adresse de votre serveur et admirez votre app***

## Problèmes
### Erreur upload de fichier
Par default nginx limite la taille des uploads à 1M, si vous essayer d'uploader un fichier plus lourd, une erreur se produit.
Il faut ajouter cette ligne dans le fichier `/etc/nginx/nginx.conf` dans le contexte `server`
`client_max_body_size 2M;`