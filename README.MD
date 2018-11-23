# Ansible configuration for NodeJS APPs
## Configuration
#### Server side
- Sur serveur Debian ou Ubuntu
- Il est nécessaire d'avoir pré-configuré un accés root ssh avec les clefs associées

#### Client side
hosts file
```
# Modifier l'adresse ip dans le fichier host avec celle de votre serveur
# Il est possible d'utiliser plusieurs adresses ip simultanement (1 ligne / ip)
[web]
71.85.130.169
```

##### Lancer le script ansible
playbook.yml
- starter => Installe les packages necessaires
- deployer => Ajoute et configure un acces ssh pour l'user "deploy"
- node => Installe nvm (Aller à la section node pour finir la configuration)
- mysql => Installe un serveur mysql, créer une base de données, un user, et attribue les droits de la base de données à l'user 
- nginx => Installe et configure nginx pour un ou plusieurs environnements (avant d'executer les tasks nginx il faut avoir deploy et démarrer l'app sur le serveur via pm2, puis il faut modifier le fichier `nginx/tasks/main.yml` avec les bonnes valeurs)
```shell
ansible-playbook -i hosts playbook.yml
```
##### Node
les commandes nvm et npm ne fonctionne pas très bien avec ansible, il faut donc se connecter manuellement et executer les commandes suivantes
```shell
nvm install 8.11.3 # or latest
npm install pm2 -g
```